---
layout: default
title: Work Board
---

# Work Board

## 지금 우선순위

업데이트: 2026-04-08 18:22:01 +09:00

- startup load baseline을 같은 경로에서 계속 재현하면서 XML 비용과 파일 fan-out을 줄이는 게 1순위다.
- parser micro-opt는 하되, 진짜 시간 절감이 manifest/scene pack에서 나오는지 바로 다시 확인해야 한다.

## XML / Startup

1. range 기반 XML parser refactor 안정화
Spec: 기존 `.deasset` key/value 결과를 유지해야 한다. nested element, explicit `key/name/value`, attribute serialization, whitespace trim, entity unescape, duplicate-key failure가 모두 이전과 동일해야 한다.

2. 같은 startup capture 경로로 전후 비교 재측정
Spec: `Scenes/BistroTest.deasset` 기준으로 `startup_capture`, `Framework::loadWorld`, `XML total read+parse`, document/file count를 baseline과 같이 기록해야 한다.

3. root manifest 비용 줄이기
Spec: 지금처럼 one-file-per-entity/component/mesh fan-out를 계속 startup에서 그대로 내지 않게, root world manifest나 chunk 단위 로드 전략을 정리해야 한다.

4. runtime scene pack 경계 설계
Spec: authoring 포맷은 XML로 유지하되 runtime load path는 나중에 chunked/packed scene data를 먹을 수 있게 경계를 분리해야 한다.

## Validation

- 모든 최적화 주장은 같은 startup capture 경로에서 전후 수치가 있어야 한다.
- parser 작업은 build만 통과해서 끝나는 게 아니라 authored asset 로드 correctness 확인이 따라와야 한다.
- XML 쪽은 tiny-file fan-out과 같이 봐야 해서 parser micro-opt만으로 결론 내리면 안 된다.

## 지금 보이는 관찰

- 측정상 병목은 단일 exotic render path가 아니라 XML parse와 문서 수에 가깝다.
- 현재 baseline에서 XML read+parse만 2.4초 수준이라 parser와 구조 양쪽을 같이 봐야 한다.
- 큰 절감이 scene pack 쪽에서 나올 가능성이 높기 때문에 parser 정리 직후 바로 다음 구조 작업으로 넘어갈 준비가 필요하다.

## 유지할 원칙

- 측정 유지
- correctness 먼저
- parser와 구조를 분리해서 보되 둘 다 숫자로 판단

## Engine Devlogs

{% assign engine_posts = site.posts | where_exp: "post", "post.tags contains 'engine'" %}
{% for post in engine_posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) - {{ post.date | date: "%Y-%m-%d %H:%M" }}
{% endfor %}
