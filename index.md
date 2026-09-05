---
layout: default
title: 글
---
<div class="eyebrow">{{ site.tagline }}</div>
<h1>실무에서 확인한 것만 씁니다</h1>
<p>PLC·서보·로봇을 한 라인에서 연동할 때 생기는 문제를 <strong>증상 → 원인 파라미터 → 검증 방법 → 출처</strong> 순서로 기록합니다. 모든 글 끝에 발행 전 검증표가 있습니다.</p>

{% include subscribe.html %}

<h2>최근 글</h2>
{% if site.posts.size == 0 %}
<p class="m">첫 글 준비 중 — 시리즈 1 "XG5000 ↔ 서보 연동 실전" 1편이 먼저 올라옵니다.</p>
{% endif %}
<ul class="post-list">
{% for post in site.posts %}
  <li>
    <a class="t" href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <div class="m">{{ post.date | date: "%Y-%m-%d" }}{% if post.series %} · {{ post.series }} {{ post.part }}편{% endif %}{% if post.verified %} · 검증 완료{% endif %}</div>
  </li>
{% endfor %}
</ul>
