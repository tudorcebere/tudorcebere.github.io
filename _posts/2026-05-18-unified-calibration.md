---
title: 'Open Thoughts on: A Unifying Theory of Distance from Calibration'
layout: post
tags:
  - Machine Learning
  - Model Calibration
  - Paper Notes
---

<!--more-->

Recently I was hoping to quantify if my classifier has the appropriate amount of confidence in its prediction given the data. Ideally, I wanted to get a guarantee on the worst-case misscalibration it could have. I was surprised that there was not a lot of consesus on how to measure calibration historically.

Jarosław Błasiok et al. {% sidenote 6 'Błasiok, Jarosław, et al. "A unifying theory of distance from calibration." Proceedings of the 55th Annual ACM Symposium on Theory of Computing. 2023.' %} starts by demonstrating exactly why some of the existing metrics, like Expected Calibration Error (ECE) are flawed. I agree that in the finite-sample regime, it doesn't seem possible to properly estimate ECE without relying on the discretization or binning of the output of our predictor $$ f $$. When reviewing the literature on this, I was indeed worried that the quality of the calibration score would end up depending quite a lot on the chosen binning strategy. 

They also give a great example of a predictor that is perfectly calibrated under ECE but useless utility-wise and, more interesting, non-trivial predictors take an penalty in their calibration score. The authors convinced me that ECE is structurally flawed and hard to estimate, and I didn't expect much from its heuristic variants up to this point. They continue by promising to create a rigorous setup to measure calibration, and then propose new algorithms and recover some of the existing ones, proving that they are proper. 

First, they introduce the *true distance to calibration*, which measures the $$ l_1 $$ distance between our classifier $$ f $$ and the closest perfectly calibrated predictor $$ g $$. Some initial thoughts here (which will probably be addressed by the time I finish the paper): Why $$ l_1 $$ and not some divergence such as KL? {% sidenote 1 'The authors show in Lemma 4.4 that the set of consistent calibration measures is identical across all bounded $$ l_p $$ metrics because they are polynomially related. Thus, $$ l_1 $$ is a robust and convenient choice, whereas KL divergence is not a true metric and can blow up to infinity.' %}  The idea of the "closest" calibrated predictor suggests there can be *many* perfectly calibrated predictors. Up until now, I hadn't spent much time thinking about this, but I guess it makes a lot of sense.  However, the most pressing question is: how can we know $$ g $$ in practice? (Otherwise, we would most likely just use it directly). {% sidenote 2 'The paper formalizes a separation between *Sample Access* (knowing the true underlying domain distribution) and *Prediction-Only Access* (just having the joint distribution of predictions and labels). The true distance to calibration requires Sample Access, which we almost never have in practice.' %} And tehn, while this $$ g $$ is calibrated, does that say anything about its usefulness? How do we dodge the trap of the previous example? In other words, maybe we dont care about the distance to a calibrated-but-useless classifier.

To address these estimation challenges, they propose a set of requirements for calibration proxy estimators: *completeness* (if the classifier is perfectly calibrated, the distance should be 0) and *soundness* (if it's not, there is a positive distance). 
They proceed by discussing that a *consistent* calibration measure must be polynomially tied to quantities related to the true calibration distance.


I think this is a sensible way of defining it, as it allows you to introduce a way of comparing different "common-sense" estimators. My own nitpicking here is that it doesn't necessarily give you a strict ordering between metrics in practice, but maybe that's not strictly necessary. {% sidenote 3 'They prove an information-theoretic barrier here: under the Prediction-Only Access model, it is impossible to get an approximation better than quadratic to the true distance.' %} With the formalism cleared up and a solid intuition for why the true distance is uncomputable directly, they seek to introduce computable quantities that are consistently related to it. 

They first introduce the Interval Calibration estimator (intCE), which is essentially standard binned-ECE but with a penalty for a wide selection of bins. They show that this penalty introduces a proper, consistent quadratic approximation of the true distance to calibration.

A second approach is Smooth Calibration (smCE), which they show is equivalent (up to constant factors) to the minimal amount of changes required in our predictions to achieve perfect calibration. They (re?)introduce the idea of *weighted calibration*, where different partitions suffer some reweighting based on a function $$ w $$ drawn from a larger set of weighting functions $$ W $$. The calibration error is the maximum penalty assigned by any $$ w $$ in this family.  For their smooth calibration technique, $$ W $$ is the family of all bounded 1-Lipschitz functions. {% sidenote 4 'It can actually be computed efficiently via a linear program with $$ \mathcal{O}(n) $$ variables in the number of datapoints.' %}

Then they proceed to introduce another family $$ W $$, where $$ W $$ is in a Reproducing Kernel Hilbert Space (RKHS), defining the Kernel Calibration Error (kCE). They show that using a Laplace kernel introduces a valid, theoretically consistent calibration error that is also efficiently computable. {% sidenote 5 'They also show the the Gaussian kernel fails the requirements for a proper calibration metric.' %} 

This paper was a gem to read, as it cleanly introduces a methodology to measure the quantity of interest that I was initially going after, calibration. How I will end up using this to achieve my guarantee on the worst-case misscalibration is yet to be discovered. 