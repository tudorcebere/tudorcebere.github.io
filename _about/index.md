---
layout: full-width
title: About
---
<h1 class="profile-name">Tudor Cebere</h1>

<div class="about-container">
  <div class="profile-photo-container">
    <img src="{{ site.baseurl }}/assets/img/tudor.jpg" alt="My Photo" class="profile-photo">
  </div>
  <div class="profile-text">
    <p>👋 I am third-year PhD student at Inria 🇫🇷, supervised by Aurélien Bellet. During my PhD, I visited the Vector Institute 🇨🇦 hosted by Nicolas Papernot and Harvard hosted by the OpenDP group.</p>
    <p><strong>Research & Interests</strong>: I am interested in differential privacy (DP) and its applications in making useful machine learning models with privacy guarantees.</p>
  </div>
</div>



<h1 class="publications-section-title">Publications</h1>

<!-- Publications timeline will follow below -->
<div class="publications-timeline">
  <table>
    {% assign grouped_pubs = site.data.publications | group_by: "year" | sort: "name" | reverse %}
    {% for group in grouped_pubs %}
      {% assign num_items = group.items | size %}
      {% for pub in group.items %}
        {% if forloop.first %}
          <tr>
            <td class="year-cell" rowspan="{{ num_items }}">{{ group.name }}</td>
            <td class="paper-cell">
              <div class="paper-entry">
                <!-- Paper Title -->
                <a href="{{ pub.link }}" target="_blank" class="paper-title">
                  <strong>{{ pub.title }}</strong>
                </a>
                <!-- Meta information: Authors, Venue (if any), and then Publication Type -->
                {% assign authors_formatted = pub.authors | replace: "Tudor Cebere", "<span class='underline-author'>Tudor Cebere</span>" %}
                <span class="paper-meta">
                  &nbsp;&ndash;&nbsp;{{ authors_formatted | safe }}
                  {% if pub.venue and pub.venue != "" %}
                    &nbsp;&ndash;&nbsp;<span class="venue">{{ pub.venue }}</span>
                  {% endif %}
                  &nbsp;&ndash;&nbsp;<span class="pub-tag {{ pub.type | downcase }}">{{ pub.type }}</span>
                </span>
              </div>
            </td>
          </tr>
        {% else %}
          <tr>
            <td class="paper-cell">
              <div class="paper-entry">
                <a href="{{ pub.link }}" target="_blank" class="paper-title">
                  <strong>{{ pub.title }}</strong>
                </a>
                {% assign authors_formatted = pub.authors | replace: "Tudor Cebere", "<span class='underline-author'>Tudor Cebere</span>" %}
                <span class="paper-meta">
                  &nbsp;&ndash;&nbsp;{{ authors_formatted | safe }}
                  {% if pub.venue and pub.venue != "" %}
                    &nbsp;&ndash;&nbsp;<span class="venue">{{ pub.venue }}</span>
                  {% endif %}
                  &nbsp;&ndash;&nbsp;<span class="pub-tag {{ pub.type | downcase }}">{{ pub.type }}</span>
                </span>
              </div>
            </td>
          </tr>
        {% endif %}
      {% endfor %}
    {% endfor %}
  </table>
</div>


