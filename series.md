---
layout: default
title: 시리즈
permalink: /series/
---
<div class="eyebrow">Series</div>
<h1>시리즈</h1>

## 1. XG5000 ↔ 서보 연동 실전

아날로그 출력으로 서보 속도·토크 지령을 내려보내는 구성에서 스케일·램프·토크 제한·진동 문제를 파라미터 수준까지 다룹니다. 4편 완결 후 "PLC–서보 아날로그 연동 점검표 + 스케일링 FB 템플릿"으로 묶습니다.

{% assign s1 = site.posts | where: "series", "XG5000 ↔ 서보 연동 실전" | sort: "part" %}
{% if s1.size == 0 %}
| 편 | 제목 | 상태 |
|---|---|---|
| 1 | 아날로그 출력 → 서보 지령 스케일이 어긋나는 5가지 이유 | 준비 중 |
| 2 | 속도 지령 램프와 히스테리시스 — PLC에 둘까 서보에 둘까 | 예정 |
| 3 | 속도 제어 모드에서 T-REF는 지령이 아니라 제한이다 | 예정 |
| 4 | 트레이스 파형으로 진동 원인 3분류 | 예정 |
{% else %}
<ul class="post-list">{% for p in s1 %}<li><a class="t" href="{{ p.url | relative_url }}">{{ p.part }}편. {{ p.title }}</a><div class="m">{{ p.date | date: "%Y-%m-%d" }}</div></li>{% endfor %}</ul>
{% endif %}
