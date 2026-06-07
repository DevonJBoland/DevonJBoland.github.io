---
layout: page
permalink: /publications/
title: publications
description: Publications in reverse chronological order.
nav: true
nav_order: 1
---

{% include bib_search.liquid %}

<div class="publications">

  {% bibliography -f papers --group_by year %}

</div>
