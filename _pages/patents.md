---
layout: page
permalink: /patents/
title: patents
description: US patents and pending applications.
nav: true
nav_order: 3
---

<!-- _pages/patents.md -->

<h2>Issued</h2>

<div class="publications">

{% bibliography --file patents_issued --group_by none %}

</div>

<h2>Pending Applications</h2>

<div class="publications">

{% bibliography --file patents_pending --group_by none %}

</div>
