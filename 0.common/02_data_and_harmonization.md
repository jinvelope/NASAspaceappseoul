# 02. 데이터 & 하모나이제이션 분석

> 공식 Resources 탭(2026-11-13 공개 예정) 전 사전 조사 내용. 수치·URL은 공개 자료로 재확인할 것 (✅ = 확인 필요 항목).

## 1. 센서 비교: MODIS vs VIIRS

| 항목 | MODIS (Terra / Aqua) | VIIRS (S-NPP / NOAA-20 / NOAA-21) |
|---|---|---|
| 기록 시작 | Terra 2000-11, Aqua 2002-07 | S-NPP 2012-01, NOAA-20 2020-01, NOAA-21 2024 ✅ |
| 화재 제품 | MCD14 (Collection 6.1) | VNP14IMG / VJ114IMG / VJ214IMG (375 m I-band) |
| 공간 해상도 | **1 km** (nadir) | **375 m** (I-band) |
| 적도 통과 시각 | Terra ~10:30 / 22:30, Aqua ~13:30 / 01:30 | 모두 ~13:30 / 01:30 |
| 신뢰도 필드 | 0–100 (%) | low / nominal / high |
| 밝기온도 필드 | `brightness`(21/22채널), `bright_t31` | `bright_ti4`, `bright_ti5` |
| 상태 | 수명 종료 단계 (궤도 표류, 운영 종료 진행) ✅ | 현재 주력, JPSS 시리즈로 지속 |

### 왜 직접 비교가 안 되는가
1. **해상도 차이** — VIIRS 375 m는 작은 불(농업 소각 등)까지 잡아 **같은 화재라도 VIIRS 탐지 수가 수 배 많음**. 큰 화재 하나가 MODIS 1픽셀 = VIIRS 여러 픽셀로 쪼개짐.
2. **민감도 차이** — 탐지 임계값·알고리즘 상이 → 소규모·저온 화재 탐지율 차이.
3. **관측 시각/횟수 차이** — Terra(오전) 포함 여부, 위성 개수 변화(2012, 2020, 2024년 VIIRS 추가)에 따라 "하루 관측 기회" 자체가 바뀜. 화재는 일주기(오후에 강함)가 뚜렷해 관측 시각이 탐지 수에 직접 영향.
4. **궤도 표류** — 말기 Terra/Aqua는 관측 시각이 점점 변해 같은 MODIS 안에서도 비일관성 발생.
5. **스캔 각도** — 가장자리(off-nadir) 픽셀은 커져서 중복 탐지/누락 발생 (`scan`, `track` 필드로 픽셀 크기 확인 가능).

→ 단순히 개수를 이어붙이면 **2012년(VIIRS 도입) 시점에 가짜 증가 추세**가 생김. 이게 바로 해결해야 할 핵심.

## 2. 데이터 소스: NASA FIRMS

- 포털: https://firms.modaps.eosdis.nasa.gov/
- 아카이브 다운로드(대용량, 기간·지역 지정 요청): https://firms.modaps.eosdis.nasa.gov/download/
- 국가별 연도 CSV (빠른 시작용) ✅: https://firms.modaps.eosdis.nasa.gov/country/
- API (MAP_KEY 무료 발급): https://firms.modaps.eosdis.nasa.gov/api/
  - Area API 형식: `/api/area/csv/{MAP_KEY}/{SOURCE}/{west,south,east,north}/{DAY_RANGE}/{DATE}`
  - SOURCE 예: `MODIS_SP`, `MODIS_NRT`, `VIIRS_SNPP_SP`, `VIIRS_SNPP_NRT`, `VIIRS_NOAA20_NRT`, `VIIRS_NOAA21_NRT`
  - 한 요청당 조회 일수 제한(1–10일 수준) 및 MAP_KEY 호출량 제한 존재 ✅ → **20년치 이력은 API가 아닌 아카이브 다운로드로 사전 확보**해야 함
- 시각 확인용: NASA Worldview https://worldview.earthdata.nasa.gov/

### 주요 컬럼 (CSV)
| 컬럼 | 설명 | 사용처 |
|---|---|---|
| `latitude`, `longitude` | 픽셀 중심 좌표 | AOI 필터링, 그리드화 |
| `acq_date`, `acq_time` | 관측 일자/시각(UTC) | 월·연 집계, 낮/밤 분리 |
| `satellite` / `instrument` | 위성/센서 | 하모나이즈 그룹 키 |
| `confidence` | 신뢰도 | 품질 필터 |
| `frp` | 화재 복사 에너지 | 강도 기반 보정·지표 |
| `scan`, `track` | 픽셀 크기 | 해상도 보정 |
| `daynight` | D/N | 관측 조건 분리 |
| `type` (SP 제품) | 0=식생화재, 1=화산, 2=기타 고정열원, 3=해상 | **0만 사용** (가스플레어 등 제거) |

### 데이터 규모 감각
- 전 지구 VIIRS 375 m는 연간 수천만 건 수준 → 전 지구 원시 데이터를 브라우저로 보내는 건 불가.
- **전략: 사전 그리드 집계(예: 0.1°×0.1° 또는 0.25°) × 월 단위 카운트 테이블**을 만들고, 원시 CSV 다운로드는 작은 AOI에 한해 제공.

## 3. 전처리 파이프라인 (공통)

1. **품질 필터**: MODIS `confidence ≥ 30`(또는 ≥50), VIIRS `low` 제외 (임계값은 실험으로 결정)
2. **비식생 열원 제거**: `type == 0`만, 또는 고정 열원 마스크(가스플레어·화산·공장) 적용
3. **센서 그룹 정의**: `MODIS-Terra`, `MODIS-Aqua`, `VIIRS-SNPP`, `VIIRS-NOAA20`, `VIIRS-NOAA21`
4. **관측 시각 일관성**: 기본안은 오후 궤도(Aqua ↔ S-NPP, 둘 다 ~13:30)를 기준선으로 사용
5. **그리드 집계**: (grid_cell, year, month, sensor) → count, sum(frp), fire_days

## 4. 하모나이제이션 기법 후보

핵심 아이디어: **두 센서가 동시에 운영된 중첩 기간(2012-01 ~ MODIS 종료)** 을 "교정 데이터"로 사용해 MODIS 값을 VIIRS 등가값으로(혹은 그 반대로) 변환.

| # | 기법 | 방법 | 장점 | 단점 | 난이도 |
|---|---|---|---|---|---|
| A | **비율 보정 (Ratio scaling)** | 셀·월별 `k = VIIRS_count / MODIS_count`를 중첩기간에서 추정, 2000–2011 MODIS × k | 단순, 설명 쉬움 | 화재 적은 셀에서 불안정 | ★ |
| B | **회귀 보정 (Linear/log-linear)** | `log(V+1) = a + b·log(M+1)` 를 지역(대륙/바이옴/토지피복)별 적합 | 비선형성 반영, 불확실성 계산 가능 | 층화 기준 설계 필요 | ★★ |
| C | **분위수 매핑 (Quantile mapping)** | MODIS 분포를 VIIRS 분포 분위수에 맞춤 (기후 모델 bias correction 기법) | 분포 전체를 맞춤, 극값 보존 | 데이터 많이 필요 | ★★ |
| D | **존재/부재 기반 지표 (Fire-days)** | 각 그리드 셀에서 "화재 탐지된 날" 여부(0/1)로 전환 → 해상도 영향 완화 | 픽셀 분할 문제 거의 제거, 아주 견고 | 화재 "개수" 정보 손실 | ★ |
| E | **MODIS 등가 다운샘플링** | VIIRS 375 m 탐지를 1 km 격자로 병합해 MODIS처럼 셈 | 물리적으로 직관적 | 민감도 차이는 남음 | ★★ |
| F | **ML 보정 (GBM 등)** | 입력: MODIS count, FRP, 토지피복, 위도, 월 → VIIRS count 예측 | 정확도 높을 수 있음 | 과적합·설명성 문제, 시간 소모 | ★★★ |

### 추천 조합
- **E(1 km 병합) + B(층화 log-linear 회귀)** 를 메인으로,
- **D(fire-days)** 를 견고성 확인용 보조 지표로,
- 층화 기준: **대륙/바이옴 또는 MODIS Land Cover(MCD12Q1) 대분류 × 계절** ✅
- 결과는 최종적으로 **"VIIRS-equivalent hotspot count"** 단위로 통일 (현재·미래 데이터가 VIIRS이므로 VIIRS 기준이 자연스러움)

### 검증 방법 (발표에서 반드시 보여줄 것)
1. **교차검증**: 중첩 기간을 시간으로 분할(예: 2012–2017 학습 / 2018–2022 검증)해 보정된 MODIS vs 실제 VIIRS 비교 → R², MAE, 편향
2. **단절 테스트**: 보정 전/후 장기 시계열에서 2012년 step change 유무 (예: Pettitt test 또는 시각적 비교)
3. **계절성 보존**: 보정 전후 월별 패턴(피크 월)이 동일한지 확인
4. **사례 검증**: 알려진 대형 화재 시즌(예: 2019–2020 호주, 2020 브라질 판타나우, 아르헨티나 코리엔테스 2022 등)에서 이상치가 제대로 튀는지

## 5. 연소 캘린더 & 지표 정의

| 지표 | 정의 |
|---|---|
| 월별 핫스팟 수 | AOI 내 harmonized count, (year, month) |
| 기후값(baseline) | 기준기간(예: 2003–2022) 같은 달의 **중앙값** (원문이 "historical median" 언급) |
| 이상치 | `(현재 − 중앙값) / 중앙값` (%) 또는 백분위 순위, 또는 robust z-score `(x − median) / MAD` |
| 피크 시즌 | 기후값 기준 상위 N개월 |
| 연간 합계/순위 | 연도별 총합 및 전체 기간 중 순위 |
| (선택) FRP 합계 | 강도 관점 보조 지표 |

**캘린더 시각화 형태**: 행 = 연도(2001→현재), 열 = 1–12월, 셀 색 = 핫스팟 수(로그 스케일) 또는 이상치(발산형 팔레트). 우측에 연간 합계 막대, 하단에 월별 기후값 막대.

## 6. 열린 질문 (Resources 공개 후 확인)

- [ ] 공식 Resources에 CONAE 제공 데이터/예시 캘린더가 있는가?
- [ ] 권장 MODIS/VIIRS 컬렉션 버전 및 기준 기간 명시 여부
- [ ] 2026년 현재 Terra/Aqua MODIS의 정확한 운영 상태 및 FIRMS 지속 제공 여부
- [ ] NOAA-21 SP(아카이브) 제공 여부
- [ ] 목표 AOI 예시 (아르헨티나 등 특정 지역 강조 여부)
