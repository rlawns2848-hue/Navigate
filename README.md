# 해상교통량 측정 및 안전진단 최소경로 예측 시스템

국내 무역항 31곳을 대상으로 **실시간 해상교통량과 사고위험 예보를 반영한 안전 항로**를 계산하는 웹 서비스입니다.
출발항과 도착항을 고르면 거리만 따진 **최소경로**와 위험을 피해 가는 **안전경로**를 함께 보여주고, 두 경로의 거리·시간·위험 노출을 비교합니다.

> 4인 팀 프로젝트 (2026.10.20 ~ 2026.11.12) · 그린AI캠퍼스 평생교육원
> 이 저장소는 팀 저장소 [`ung-Jin/Navigate`](https://github.com/ung-Jin/Navigate) 의 fork 입니다.

---

## 담당 범위

**경로 탐색 전체 (`route` 패키지)를 맡았습니다.**

📁 **[`src/main/java/com/green/Navigate/route`](https://github.com/rlawns2848-hue/Navigate/tree/dev/Navigate/src/main/java/com/green/Navigate/route)** ← 이 폴더 전체가 제가 만든 코드입니다

| 파일 | 하는 일 |
|---|---|
| `service/SeaGrid.java` | 해안선을 500m 격자로 변환 · 항구↔바다 칸 연결 · BFS 로 고립 해역 제거 |
| `service/AStar.java` | A\* 경로 탐색 · 대각선 통과 제한 · 격자 경로를 직선으로 펴기 |
| `service/RiskMap.java` | 교통·사고위험 예보·기상을 0~1 로 맞춰 칸별 위험도로 조립 |
| `service/RouteService.java` | 최소경로·안전경로 계산 · 거리/시간/위험 측정 · 경로 캐싱 |
| `util/GeoUtil.java` | 하버사인 거리 계산 |
| `controller/RouteApiController.java` | `GET /api/route` |
| `dto/RouteDTO.java` · `dto/RoutePathDTO.java` | 응답 구조 |
| `static/js/route.js` · `route-grid.js` | 지도에 두 경로 그리기 · 비교표 렌더링 |

🔗 **[팀 저장소에서 제 커밋만 보기](https://github.com/ung-Jin/Navigate/commits/dev?author=rlawns2848-hue)**

---

## 결과

```
육지 통과율      3~16%  →  0.136%      격자 2.5km → 500m, 통과 기준 바다 30% → 80%
위험 노출        평균 -22.6%            무역항 30곳 전수 조합 435쌍 기준
경로 탐색 속도    4.4배                  부산항→목포항 93,374칸 vs 다익스트라 413,889칸 (경로 동일)
가중치 판별력     79.8%  →  85.6%        조합 11가지를 사고다발 26칸으로 채점해 선정
```

---

## 동작 흐름

```
1. 자료 수집     KOMSA MTIS (실시간 선박·사고위험 예보·사고다발) · Open-Meteo (항구 72시간 예보)
                 10분 / 1시간 / 30분 주기

2. 바다 격자     OpenStreetMap 해안선 → 1400 x 1600 격자 (500m)
                 칸마다 바다 비율 저장, 80% 이상만 통과 가능

3. 위험도 조립    교통 혼잡 · 사고위험 예보 · 파고/풍속을 각각 0~1 로 변환
                 위험도 = 0.50 x 교통 + 0.30 x 예보 + 0.20 x 기상

4. 경로 탐색     간선 비용 = 거리 x (1 + α x 위험도)
                 α = 0  →  최소경로      α = 3  →  안전경로
```

### 설계에서 신경 쓴 것

- **격자 해상도를 MTIS 칸에 맞췄습니다.** `0.025 ÷ 0.005 = 5` 로 정확히 나누어떨어져, 위험도를 가져올 때 칸 번호를 5로 나누기만 하면 됩니다.
- **위험을 거리로 환산했습니다.** 거리와 안전은 동시에 최적화할 수 없어, 위험도 1인 칸은 거리가 4배로 계산되게 두고 A\* 가 알아서 저울질하게 했습니다. 피할 칸을 따로 지정하는 임계값은 두지 않았습니다.
- **항구 둘레 5km 는 할증을 깎습니다.** 항구 앞이 위험하다고 입항을 안 할 수는 없기 때문입니다. 위험 점수 자체는 그대로 두고 경로를 고를 때의 비용만 줄였습니다.
- **내륙 항만은 격자가 걸러냅니다.** 서울항은 한강 안쪽이라 15km 안에 통과 가능한 바다 칸이 없습니다. 항구 이름을 코드에 적는 대신 격자로 판정해, 다른 내륙 항만이 추가돼도 코드 수정이 필요 없습니다.

---

## 작업 기록

작업일지를 Notion 에 남겼습니다.

| 날짜 | 내용 |
|---|---|
| 10/19 | [주제 파악 — 기존 대시보드 읽기](https://scratched-ceramic-15c.notion.site/10-19-41090bcc7b0781be8dd4d8e51f7578bc) |
| 10/20 | [역할 분담과 사고 데이터 통합](https://scratched-ceramic-15c.notion.site/10-20-41090bcc7b07811ca586c4186a2b76b8) |
| 10/26 | [경로 작업 시작 — 직선부터 연결하기](https://scratched-ceramic-15c.notion.site/10-26-41090bcc7b0781bd9f58e399e96dd606) |
| 10/29 | [바다·육지 격자 만들기](https://scratched-ceramic-15c.notion.site/10-29-41090bcc7b0781bb8967fc8dc08cd5bf) |
| **10/30** | **[격자 2.5km → 500m — 검증을 잘못했다](https://scratched-ceramic-15c.notion.site/10-30-2-5km-500m-41090bcc7b07813db085c79248184e70)** |
| 11/03 | [위험도를 반영한 안전경로](https://scratched-ceramic-15c.notion.site/11-03-41090bcc7b0781e695decd5ab7f377b9) |
| 11/05 | [경로 수정 · 시각별 예측 · 가중치 검증](https://scratched-ceramic-15c.notion.site/11-05-41090bcc7b0781258f66fc6c6efc3be5) |

### 가장 크게 배운 것

격자 위에서 항구 30곳의 모든 조합 435쌍을 검사해 **"전부 통과"** 를 받았습니다.
그런데 화면을 보니 경로가 섬을 그대로 지나가고 있었습니다.

```
검증 기준 = 내가 만든 2.5km 격자
검증 대상 = 그 격자로 만든 경로
            ↑ 같은 자료로 채점하니 당연히 통과
```

원본 해안선(110m)으로 다시 재니 경로의 **3~16% 가 육지 위**였습니다.
격자를 500m 로 줄이고 통과 기준을 80% 로 올려 0.136% 까지 내렸습니다.

> 스스로 만든 자로 자기 결과를 재면 항상 맞다고 나옵니다. 자는 밖에서 가져와야 합니다.

---

## 기술 스택

| | |
|---|---|
| **Backend** | Java 17 · Spring Boot 4.0.8 · MyBatis 4.0.1 · Thymeleaf · Lombok |
| **Database** | MariaDB |
| **Frontend** | JavaScript · Leaflet · OpenStreetMap |
| **격자 생성** | Python · NumPy |
| **External API** | KOMSA MTIS · Open-Meteo · 국립해양조사원 CCTV |
| **협업** | Git (dev / feature 브랜치 · Pull Request) |

---

## 실행 방법

### 1. 데이터베이스

```bash
mysql -u user -p navigate < Navigate/sql/ports.sql
```

### 2. `Navigate/.env` 작성

인증키와 접속 정보는 깃에 올리지 않습니다. 아래 네 항목이 필요합니다.

```properties
DB_HOST=
DB_PASSWORD=
MTIS_KEY=          # 공공데이터포털 - 한국해양교통안전공단 MTIS
CCTV_BASE=         # 국립해양조사원 항만 영상 (이용조건상 비공개)
```

### 3. 실행

```bash
cd Navigate
./gradlew bootRun
```

`http://localhost:8080` 에서 확인할 수 있습니다.

---

## 팀 구성

| 담당 | 범위 |
|---|---|
| 이웅진 | 공통 기반 — 서버·지도 뼈대, 팀 코드 통합, 항만 CCTV |
| 최준영 | 교통과 위험 예보 — 실시간 선박 위치, 사고위험 예보, 사고·좌초 다발 지점 |
| 박성인 | 날씨와 데이터 검토 — 항구 72시간 예보, 기상 패널·그래프, 학습 데이터 점검 |
| **김성준** | **경로 예측 — 바다 격자, 최소·안전 경로, 위험도 조립** |

격자와 위험도는 팀원이 수집한 자료를 받아 제가 조립했고, 세 자료를 섞는 비중은 팀에서 함께 정했습니다.
