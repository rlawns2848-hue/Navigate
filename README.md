# 해상교통 안전경로 예측 시스템

> 출발·도착 항구를 고르면 **거리만 보는 최소경로**와 **교통·사고위험·날씨를 반영한 안전경로**를 지도에 함께 그려 주는 출항 전 계획 도구

![Java](https://img.shields.io/badge/Java_17-437291?style=flat-square&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![MyBatis](https://img.shields.io/badge/MyBatis-000000?style=flat-square)
![MariaDB](https://img.shields.io/badge/MariaDB-003545?style=flat-square&logo=mariadb&logoColor=white)
![Leaflet](https://img.shields.io/badge/Leaflet-199900?style=flat-square&logo=leaflet&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)

## 한눈에 보기

| 구분 | 내용 |
| --- | --- |
| 팀·기간 | Hello_World 4인 · 2026.10.17 ~ 11.12 (4주) |
| **내 담당** | **`route` 패키지 전체 — 격자 제작부터 화면에 그리기까지** |
| 기술 | Java 17 · Spring Boot · MyBatis · MariaDB · Leaflet · Python(numpy) |
| 원본 저장소 | [`ung-Jin/Navigate`](https://github.com/ung-Jin/Navigate) 의 fork · [내 커밋](https://github.com/ung-Jin/Navigate/commits/dev?author=rlawns2848-hue) |
| 작업일지 | [Notion](https://scratched-ceramic-15c.notion.site/40090bcc7b0780409d99e0e6757d8e30) |

바다에는 차선이 없고 파고·풍속·교통량이 시시각각 바뀝니다. 그래서 짧은 길과 안전한 길이 같지 않습니다.
실시간 내비게이션이 아니라, **예보(오늘~모레)를 보고 출항 전에 계획을 세우는** 도구입니다.

## 내가 만든 것

| | 하는 일 | 파일 |
| --- | --- | --- |
| **바다 격자** | 해안선을 500m 칸으로 자르고, 큰 바다와 끊긴 만을 걸러냄 | [`SeaGrid`](Navigate/src/main/java/com/green/Navigate/route/service/SeaGrid.java) |
| **위험도 조립** | 교통·사고위험 예보·기상을 각각 0~1 로 맞춰 칸마다 한 점수로 | [`RiskMap`](Navigate/src/main/java/com/green/Navigate/route/service/RiskMap.java) |
| **경로 탐색** | A\* · 대각선 통과 제한 · 계단 경로를 직선으로 펴기 | [`AStar`](Navigate/src/main/java/com/green/Navigate/route/service/AStar.java) |
| **최소·안전 경로** | 같은 A\* 를 α만 바꿔 두 번 · 거리/시간/위험 측정 · 캐싱 | [`RouteService`](Navigate/src/main/java/com/green/Navigate/route/service/RouteService.java) |
| **화면** | 지도에 두 경로를 그리고 거리·시간·위험을 비교표로 | [`route.js`](Navigate/src/main/resources/static/js/route.js) |

<details><summary>팀원 담당</summary>

| 담당 | 범위 |
| --- | --- |
| 이웅진 | 공통 기반 — 서버·지도 뼈대, 팀 코드 통합, 항만 CCTV |
| 최준영 | 교통과 위험 — 실시간 선박 위치, 사고위험 예보, 사고·좌초 다발 지점 |
| 박성인 | 날씨와 검토 — 항구 72시간 예보, 기상 패널·그래프, 학습 데이터 점검 |

</details>

## 화면

![경로 탐색 — 군산항 → 제주항](docs/route-screen.png)

회색 점선이 최소경로, 파란 실선이 안전경로입니다. 노랑·주황·빨강 칸이 위험도입니다.

최소경로는 완도항 앞의 붉은 칸을 그대로 지나가고, 안전경로는 그 서쪽으로 비켜 갑니다.
**12km(3.8%) 더 돌아가는 대신 위험 노출이 25.2 → 14.0 으로 44.4% 줄었습니다.** 시간은 30분 늘었습니다.

## 핵심 코드

| 주제 | 판단 | 코드 |
| --- | --- | --- |
| 계단 경로 펴기 | 앞에서부터가 아니라 **뒤에서부터** 이을 지점을 찾는다. 앞에서부터 늘리면 눈앞에서 막힐 때 거기서 끊겨 더 멀리 이을 기회를 놓친다 — 울산항→제주항에서 13km 차이 | [`AStar#L228`](Navigate/src/main/java/com/green/Navigate/route/service/AStar.java#L228) |
| 직선이 바다인지 검사 | 일정 간격으로 짚지 않고 **직선이 지나는 칸을 하나씩 빠짐없이** 따라간다. 간격으로 짚으면 모서리를 스치는 0.3km 육지 통과를 놓친다 | [`AStar#L262`](Navigate/src/main/java/com/green/Navigate/route/service/AStar.java#L262) |
| 대각선 이동 | 양옆 두 칸이 **모두 바다일 때만** 허용. 아니면 두 육지 사이를 비스듬히 뚫고 지나간다 | [`AStar#L126`](Navigate/src/main/java/com/green/Navigate/route/service/AStar.java#L126) |
| 항구 둘레 5km | 위험 가중치를 거리에 비례해 줄인다. 항구 앞은 배가 몰려 위험도가 높은데 그대로 두면 항만을 빙 돌아 진입한다. **경로를 고를 때만 적용되고 지도 위험도는 그대로** | [`AStar#L183`](Navigate/src/main/java/com/green/Navigate/route/service/AStar.java#L183) |
| 항구를 바다 칸으로 | 그냥 통과 가능한 칸이 아니라 **큰 바다에 이어진 칸**만 고른다. 아니면 마산항처럼 2칸짜리로 끊긴 만 안쪽이 잡혀 경로를 못 찾는다 (마산항은 큰 바다까지 7.1km) | [`SeaGrid#L266`](Navigate/src/main/java/com/green/Navigate/route/service/SeaGrid.java#L266) |
| 날씨 점수 | 파고 2m 와 4m 가 2배 위험한 게 아니라 **한계를 넘으면 급격히** 위험해진다. 한계의 절반까지 0, 한계에서 1, 넘으면 1 | [`RiskMap#L233`](Navigate/src/main/java/com/green/Navigate/route/service/RiskMap.java#L233) |
| 안전경로 비용 | `거리 × (1 + α × 위험도)`, α = 3. 위험한 칸을 **막지 않고 비싸게** 만들어 A\* 가 고르게 한다 | [`RouteService#L49`](Navigate/src/main/java/com/green/Navigate/route/service/RouteService.java#L49) |
| 최소경로 캐시 | 거리만 보므로 **시각과 무관** → 출항 시각을 바꿔도 다시 계산하지 않는다 (계산 시간의 절반) | [`RouteService#L175`](Navigate/src/main/java/com/green/Navigate/route/service/RouteService.java#L175) |

## 문제 해결

**1. 내륙에 있는 무역항** — 서울항은 한강 여의도 부근이라 바다까지 29km 이고 그 사이가 전부 육지입니다. 무역항 31곳 중 서울항만 경로를 만들 수 없습니다.
**항구 이름을 코드에 적어 두지 않고 격자로 판정**했습니다. 주변 15km 안에 큰 바다에 이어진 칸이 없으면 경로 없음으로 답합니다. 나중에 다른 내륙 항만이 늘어도 그대로 걸러집니다. → [`SeaGrid#L266`](Navigate/src/main/java/com/green/Navigate/route/service/SeaGrid.java#L266)

**2. 뒤늦은 응답이 새 결과를 덮어썼다** — '내일'을 고르고 이어서 '06시'를 고르니 **화면 라벨은 06시인데 실제로 쓴 예보는 15시**였습니다.

```
① '내일' 클릭 → 요청 A(15시) 출발
② '06시' 클릭 → 요청 B(06시) 출발
③ B 먼저 도착 → 화면에 그림
④ A 늦게 도착 → B를 덮어씀        ← 라벨과 내용이 어긋남
```

경로 계산이 몇 초 걸리고 시각마다 시간이 달라서 생긴 일입니다. **요청마다 번호를 매기고 가장 마지막 요청의 응답만 그리도록** 했습니다. → [`route.js#L25`](Navigate/src/main/resources/static/js/route.js#L25)

## 결과

![A* 와 다익스트라가 살펴본 칸](docs/astar-vs-dijkstra.png)

부산항 → 목포항에서 **두 알고리즘이 실제로 살펴본 칸**입니다 (주황색). 찾아낸 경로는 같지만,
A\* 는 93,375칸만 보고 0.82초, 다익스트라는 413,890칸을 보고 3.24초 걸렸습니다.

무역항 31곳 중 내륙인 서울항을 뺀 30곳의 모든 조합(435쌍)을 전부 돌린 결과입니다.

| 항목 | 결과 |
| --- | --- |
| 안전경로의 평균 위험점수 | 최소경로 대비 **−22.6%** (가장 크게 줄어든 구간 −63.6%) |
| 경로의 육지 통과 비율 (110m 해안선 기준) | **0.136%** — 2.5km 격자였을 때는 3~16% |
| A\* 탐색량 (부산→목포) | 93,375칸 · 0.82초 — 다익스트라는 413,890칸 · 3.24초 (**4.4배**) |
| 위험 가중치 판별력 | 사고다발 26칸을 가려내는 확률 79.8% → **85.6%** |
| 경로를 못 찾은 구간 | 없음 |

## 한계

- **육지 통과가 0% 는 아닙니다.** 0.136% 가 남아 있어 그대로 항해에 쓸 수 있는 수준이 아닙니다.
- **사고다발 26칸은 가중치를 고를 때 정답지로만** 썼고 경로 비용에는 직접 넣지 못했습니다. 좌초 87곳은 지도 표시만 합니다.
- **α = 3 과 항구 5km 는 검증하지 못한 값**입니다. 가중치(0.5·0.3·0.2)는 채점해서 골랐지만 이 둘은 그러지 못했습니다.
- 속도는 12노트 고정이라 **시간 표시에만** 쓰입니다. 경로와 위험 비교 결과는 이 값과 무관합니다.
- 수심은 2차원 격자라 깊이 축 자체가 없습니다.

## 실행 방법

1. MariaDB 에 `sql/ports.sql` 을 실행합니다.
2. 프로젝트 루트에 `.env` 를 만들고 `MTIS_KEY` · `CCTV_BASE` · `DB_HOST` · `DB_PASSWORD` 를 넣습니다.
3. 서버를 실행하고 출발·도착 항구를 고릅니다. `.env` 를 고친 뒤에는 서버를 다시 시작합니다.

> API 키와 항만 영상 주소는 이용 조건에 따라 저장소에 포함하지 않았습니다.

## 배운 점

1. **내가 만든 기준으로 검증하면 언제나 통과합니다.** 2.5km 격자로 435쌍을 돌려 육지 통과 0건이 나왔는데, 채점에 쓴 자가 제가 만든 그 격자였습니다. 110m 해안선으로 다시 재니 3~16% 가 육지 위였습니다.
2. **높은 점수가 잘한 게 아니었습니다.** 정확도 77.3% 는 전부 무사고로 찍어도 나오는 점수였습니다.
3. **기준이 없는 값도 근거를 만들 수 있습니다.** 가중치는 어디에도 기준이 없어 감으로 정했는데, 사고다발 26칸을 정답지로 삼아 11가지 조합을 채점해 골랐습니다.
 메모리에만 둡니다.

사고위험·혼잡 예보는 **3시간 간격**으로 나옵니다. 같은 구간이면 그날 처음 한 번만 받아 다시 사용합니다.

바다 격자는 2.2MB 파일을 메모리에 올려 두고, **항구 좌표만 DB 테이블에 담아 사용합니다.**
