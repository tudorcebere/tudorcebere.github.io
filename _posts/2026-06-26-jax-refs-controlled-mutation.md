---
title: 'JAX Refs: Controlled Mutation'
date: 2026-06-26
layout: post
tags:
  - JAX
  - Machine Learning
  - Systems
  - Autodiff
---

{% newthought 'JAX Refs' %} are mutable array cells. They let you read and write array state while remaining inside JAX transformations such as `jit` and `grad`. <!--more-->

A `jax.Array` is a value. In JAX, values do not change. To “update” an array, you compute another array:

```python
x = x.at[i].set(y)
```

This strictness is one reason JAX composes well with compilation, automatic differentiation, vectorization, and parallel execution. The compiler sees values and dependencies, not hidden writes.

But immutability is not always the clearest way to write a program.

{% marginnote 'ref-use-cases' 'Refs are best for operational state: logs, buffers, scratch space, and diagnostics.' %}
Sometimes the thing you want is not a new value. You want a place to put data: a loss computed deep inside a training step, a scratch buffer used inside a loop, a running statistic, a diagnostic from a backward pass, or an output buffer filled in-place.

`Ref` is JAX’s small, explicit answer to that problem. {% sidenote 'jax-refs-docs' 'JAX documentation, “Ref: mutable arrays for data plumbing and memory control”.' %}

## Values and locations

The distinction is simple:

{% marginnote 'values-cells-table' 'Table 1: arrays are values; refs are mutable locations.' %}
<div class="table-wrapper" markdown="1">

| Object | Mental model | Operation |
|:---|:---|:---|
| `jax.Array` | A value | Compute a new value |
| `jax.Ref` | A mutable location | Read or write the stored value |

{: .booktabs style="margin: 0 auto;"}
</div>

An array expression does not update its inputs:

```python
x = jnp.array([1., 2., 3.])
y = x + 1
```

The original `x` is still `[1., 2., 3.]`. The expression produced a new value, `y`.

A `Ref` is different. It names a location that contains an array value:

<div class="code-listing" markdown="1">

```python
x_ref = jax.new_ref(jnp.array([1., 2., 3.]))

x_ref[1] += 10

print(x_ref[...])  # [ 1., 12.,  3.]
```

</div>

The syntax follows NumPy indexing. Read from a ref by indexing it:

```python
x = x_ref[...]
```

Write the whole value with assignment:

```python
x_ref[...] = x + 1
```

Update a slice in-place:

```python
x_ref[i] += delta
```

The notation is standard, but the meaning is not. A `Ref` introduces controlled mutation into JAX’s staged computation model.

## Why refs exist

The usual JAX style is functional:

```python
new_state, output = f(old_state, input)
```

That style is often the right one. It makes dependencies explicit, keeps functions easy to transform, and gives the compiler a clean program to analyze.

But not all data in a program has the same status. Model parameters, optimizer state, batch inputs, and PRNG keys usually belong in the function signature. They define the computation.

Metrics, scratch space, and diagnostics are different. They are often operational. You need them, but they are not always part of the function you are trying to express.

A `Ref` gives that operational data an explicit home:

> This state is mutable, and its location is visible.

That is the compromise. Mutation is allowed, but it is attached to a particular reference, not hidden in arbitrary Python state.

## Recording losses inside a loop

Suppose we run a few steps of gradient descent and want to record the loss at each step.

<div class="code-listing" markdown="1">

```python
import jax
import jax.numpy as jnp

def mse(w, x, y):
    pred = x @ w
    return jnp.mean((pred - y) ** 2)

steps = 5
lr = 0.1
```

</div>

### Without `Ref`: carry the log through the loop

In ordinary functional JAX, the loss history must be part of the loop state.

<div class="code-listing" markdown="1">

```python
@jax.jit
def train_without_refs(w, x, y):
    losses0 = jnp.zeros((steps,))

    def body(carry, t):
        w, losses = carry

        loss, grad = jax.value_and_grad(mse)(w, x, y)
        w = w - lr * grad

        # This returns a new array value.
        losses = losses.at[t].set(loss)

        return (w, losses), None

    (w, losses), _ = jax.lax.scan(
        body,
        (w, losses0),
        jnp.arange(steps),
    )
    return w, losses
```

</div>

This is good JAX. It is explicit and transformable. But the log has become part of the loop carry. In a small example that is harmless. In a real training step, this pattern spreads: every function that computes an interesting diagnostic must return it, and every caller must forward it. The algorithm becomes harder to read because the bookkeeping travels with the state.

### With `Ref`: put the log in an output buffer

With a `Ref`, the loss history can live in a separate mutable buffer.

<div class="code-listing" markdown="1">

```python
@jax.jit
def train_with_refs(w, x, y, losses_ref):
    def body(w, t):
        loss, grad = jax.value_and_grad(mse)(w, x, y)
        w = w - lr * grad

        # The loss is diagnostic data, not optimization state.
        losses_ref[t] = jax.lax.stop_gradient(loss)

        return w, None

    w, _ = jax.lax.scan(
        body,
        w,
        jnp.arange(steps),
    )
    return w
```

</div>

The caller owns the buffer:

<div class="code-listing" markdown="1">

```python
losses_ref = jax.new_ref(jnp.zeros((steps,)))

w_final = train_with_refs(w0, x, y, losses_ref)
losses = losses_ref[...]

print(losses)
```

</div>

The structural change is the point. Without refs:

```text
(w, losses) -> body -> (w, losses)
```

With refs:

```text
w -> body -> w
```

The loop carry is now just the optimization state. The loss history is still explicit, but it lives beside the algorithm rather than inside it. That is the main use of `Ref`: separating the value flow of the computation from the data plumbing around it.

## Why `stop_gradient` appears

This line matters:

```python
losses_ref[t] = jax.lax.stop_gradient(loss)
```

The loss buffer is a diagnostic. We want to record the value, not make the mutable write part of the differentiated computation.

A useful rule is:

> If a ref is used only for metrics, logging, or diagnostics, write stopped values into it.

This keeps the mathematical function visible:

```python
w_final = train(w, x, y)
```

The ref records what happened. It does not define what the function is.

## A `Ref` is not an array

The right mental model is that references are similar to boxes where you can put values:

<div class="code-listing" markdown="1">

```python
x_ref = jax.new_ref(1.0)

try:
    jnp.sin(x_ref)   # wrong
except TypeError:
    pass

jnp.sin(x_ref[...])  # right
```

</div>

A ref is the mutable location. The array value is stored inside it.

The two basic operations are:

```python
x_ref[...]      # read the stored value
x_ref[...] = y  # replace the stored value
```

For indexed state:

```python
x_ref[i]        # read one slice
x_ref[i] = y    # write one slice
x_ref[i] += y   # update one slice
```

The compact rule is:

> Do math on `ref[...]`, not on `ref`.

The brackets are not decoration. They are the boundary between the mutable cell and the array value inside it. This is similar to dereferencing or pointer programming in imperative languages like C/C++.

## When to use refs

Use refs when the program is clearer with an explicit mutable location.

{% marginnote 'ref-boundary-table' 'Table 2: operational state can live in refs; mathematical state should usually remain an array.' %}
<div class="table-wrapper" markdown="1">

| Prefer a `Ref` for | Prefer an array for |
|:---|:---|
| Metric buffers | Model parameters |
| Temporary scratch arrays | Optimizer state |
| Output buffers | Batch inputs |
| Running statistics and logs | Return values that define the function |
| Diagnostics from compiled code | Values you want to batch, serialize, or pass around freely |

{: .booktabs style="margin: 0 auto;"}
</div>


If a value is part of the mathematical interface of the function, pass it and return it. If it is operational data that needs a stable location, a ref may be the clearer tool.

## End note

A `Ref` is a controlled mutable cell for JAX arrays.

An array is a value. A ref is a location containing a value. The compact rule is:

```python
ref[...]      # read
ref[...] = x  # write
```

Internal refs can preserve a pure interface from the caller’s perspective. Ref arguments and closed-over refs are explicit side effects. The restrictions around refs exist to rule out ambiguous aliasing and transformation behavior.

The point is not that JAX becomes imperative. The point is narrower: JAX now has a small, explicit place for mutation when mutation is the clearest way to express data plumbing or memory control.
