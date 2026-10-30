# 04. 입문자용 학습 리소스 & 로드맵

> 우주항공/원격탐사 배경지식이 없는 상태에서 이 챌린지를 수행하기 위한 학습 가이드.
> 링크는 2026-10 기준이고, ✅ 표시는 접속해서 위치가 맞는지 확인이 필요한 항목.

---

## 0. 먼저 안심할 것: 알아야 할 것 vs 몰라도 되는 것

이 챌린지는 **"우주항공 공학"이 아니라 "위성 데이터 분석 + 웹 개발"** 문제다.
로켓, 궤도역학, 위성 하드웨어는 몰라도 된다.

| 꼭 알아야 함 | 대략만 알면 됨 | 몰라도 됨 |
|---|---|---|
| 핫스팟(active fire) 데이터가 무엇인지 | 적외선으로 열을 감지하는 원리 | 궤도 계산, 위성 설계 |
| FIRMS CSV 컬럼 의미 | 화재 탐지 알고리즘 세부 수식 | 복사전달 방정식 |
| MODIS와 VIIRS의 차이 (해상도, 관측 시각) | 위성 궤도 종류(태양동기궤도) | 원시 위성 영상(L0/L1) 처리 |
| 시계열 집계, 기후값(중앙값), 이상치 | 토지피복/바이옴 개념 | HDF/NetCDF 영상 처리 (CSV로 충분) |
| 지도 위 영역 선택, 좌표계(위경도) | 좌표계 투영 | |

---

## 1. 학습 로드맵 (추천 순서)

| 단계 | 목표 | 예상 시간 | 참고 섹션 |
|---|---|---|---|
| 1단계 | 원격탐사·위성 화재 탐지가 뭔지 감 잡기 | 2–3시간 | §2, §3 |
| 2단계 | FIRMS 사이트에서 실제 데이터 보고, CSV 직접 받아보기 | 2시간 | §4 |
| 3단계 | MODIS vs VIIRS 차이 이해 (이 챌린지의 핵심) | 2–3시간 | §5 |
| 4단계 | 비슷한 기존 서비스 둘러보기 (벤치마킹) | 1시간 | §6 |
| 5단계 | 데이터 분석 기법 (시계열, 이상치, 보정) | 3–4시간 | §7 |
| 6단계 | 기술 스택 튜토리얼 (지도, 차트, 데이터 처리) | 역할별 | §8 |
| (선택) | 논문으로 깊게 이해 | 여유 시 | §9 |

---

## 2. 기초 개념: 원격탐사(Remote Sensing)

### 핵심 개념을 쉽게 설명하면
- **원격탐사**: 직접 가지 않고 멀리서(위성, 비행기) 센서로 지표를 측정하는 것.
- **전자기 스펙트럼**: 눈에 보이는 빛(가시광선) 외에 적외선 등 보이지 않는 빛도 있다. 위성 센서는 여러 "밴드(채널)"로 각 파장의 빛을 따로 측정한다.
- **열적외선 / 중적외선(TIR / MIR)**: 뜨거운 물체는 적외선을 강하게 방출한다. 불이 나면 주변보다 훨씬 밝게 보여서 탐지된다.
- **밝기온도(Brightness Temperature)**: 센서가 측정한 적외선 세기를 온도(K)로 환산한 값. FIRMS의 `brightness`, `bright_ti4` 컬럼.
- **픽셀 / 공간해상도**: 위성 이미지 한 칸이 지상에서 차지하는 크기. MODIS는 1 km, VIIRS는 375 m. 픽셀 안 일부만 타도 탐지될 수 있다(그래서 "핫스팟 1개 ≠ 화재 1건").
- **태양동기궤도**: 위성이 매일 같은 현지 시각에 같은 지역 위를 지나는 궤도. 그래서 "오전 10:30 위성", "오후 1:30 위성" 같은 표현을 쓴다.
- **Nadir / Off-nadir**: 위성 바로 아래(nadir)는 픽셀이 작고, 가장자리로 갈수록 픽셀이 커진다.

### 리소스
| 리소스 | 설명 | 링크 |
|---|---|---|
| NASA Earthdata – 원격탐사 기초 | 원격탐사 입문 글 (가장 먼저 읽기) | https://www.earthdata.nasa.gov/learn/earth-observation-data-basics/remote-sensing ✅ |
| NASA ARSET | NASA 무료 원격탐사 교육 (입문 강의, 화재 관련 강의 다수, 영상+슬라이드) | https://appliedsciences.nasa.gov/what-we-do/capacity-building/arset |
| ARSET "Fundamentals of Remote Sensing" | 완전 입문 과정 | ARSET 사이트에서 검색 ✅ |
| NASA Earth Observatory | 위성 사진과 해설 기사, 화재 관련 글 다수 | https://earthobservatory.nasa.gov/ |
| YouTube 검색어 | "remote sensing basics", "how satellites detect fires", "NASA FIRMS tutorial" | — |

---

## 3. 위성 화재 탐지 (Active Fire Detection)

### 핵심 개념
- **Active fire / Hotspot**: 위성이 지나가는 순간 **타고 있던** 불. (이미 탄 흔적은 "Burned Area"라는 별도 제품)
- **탐지 원리**: 4 µm 부근(중적외선) 밝기가 주변 픽셀보다 확 튀고, 11 µm(열적외선)와의 차이가 크면 화재로 판정하는 "문맥(contextual) 알고리즘".
- **FRP (Fire Radiative Power)**: 화재가 방출하는 에너지(MW). 화재 크기/강도 지표.
- **Confidence(신뢰도)**: 이 탐지가 진짜 화재일 확률. 구름 가장자리, 햇빛 반사, 뜨거운 사막 등이 오탐을 만든다.
- **한계**: 구름/연기 아래 불은 못 봄, 위성이 지나가지 않는 시간의 불은 못 봄, 작고 차가운 불은 놓침.
- **화재 종류**: 산불(wildfire), 농업 소각, 개간을 위한 화전, 가스 플레어(유전), 화산 → 챌린지에서는 식생 화재 위주로 분석.

### 리소스
| 리소스 | 설명 | 링크 |
|---|---|---|
| ARSET 화재 관련 교육 | "Satellite Observations for Fire" 계열 과정. 이 챌린지와 가장 직결됨 | ARSET 사이트에서 "fire" 검색 |
| UMD MODIS Fire 사이트 | MODIS 화재 제품 공식 사이트, User Guide 다운로드 | https://modis-fire.umd.edu/ |
| NASA Earth Observatory – Fire 지도 | 월별 전 지구 화재 지도 (계절성 감 잡기 좋음) | https://earthobservatory.nasa.gov/global-maps/MOD14A1_M_FIRE |
| NASA Earthdata – Wildfires 주제 페이지 | 화재 관련 NASA 데이터 총정리 | https://www.earthdata.nasa.gov/topics/human-dimensions/wildfires ✅ |

---

## 4. NASA FIRMS (이 프로젝트의 메인 데이터)

**FIRMS = Fire Information for Resource Management System.** MODIS·VIIRS 핫스팟을 CSV/Shapefile/API로 제공하는 NASA 서비스.

### 해볼 것 (실습)
1. FIRMS 지도 열기 → 한국/호주/브라질 등 지역 확대 → 점 하나 클릭해서 속성 보기
2. 왼쪽 레이어에서 MODIS / VIIRS를 각각 켜고 꺼보며 **점 개수 차이 직접 확인** (이게 챌린지 문제)
3. 국가별 연도 CSV 하나 받아서 엑셀/파이썬으로 열어보기
4. MAP_KEY 발급받고 API로 최근 데이터 받아보기

### 리소스
| 리소스 | 설명 | 링크 |
|---|---|---|
| FIRMS 메인 / 지도 | 실시간 화재 지도 | https://firms.modaps.eosdis.nasa.gov/ |
| FIRMS 아카이브 다운로드 | 과거 데이터 기간/지역 지정 요청 | https://firms.modaps.eosdis.nasa.gov/download/ |
| FIRMS 국가별 CSV | 국가·연도별 바로 다운로드 | https://firms.modaps.eosdis.nasa.gov/country/ ✅ |
| FIRMS API + MAP_KEY 발급 | API 문서 및 키 발급 | https://firms.modaps.eosdis.nasa.gov/api/ |
| Earthdata FIRMS 소개 페이지 | 데이터 속성(컬럼) 설명, FAQ | https://www.earthdata.nasa.gov/data/tools/firms ✅ |
| Earthdata 로그인 | NASA 데이터 계정 (무료) | https://urs.earthdata.nasa.gov/ |
| NASA Worldview | 위성 영상 위에 핫스팟을 겹쳐 보는 도구 (연기·구름과 비교) | https://worldview.earthdata.nasa.gov/ |

---

## 5. MODIS vs VIIRS (챌린지 핵심)

### 꼭 이해할 것
- MODIS: Terra(2000~, 오전 위성), Aqua(2002~, 오후 위성). 1 km. 오래된 기록이 강점. 수명 종료 단계.
- VIIRS: S-NPP(2012~), NOAA-20(2018 발사), NOAA-21(2022 발사). 375 m. 더 많은, 더 작은 불을 잡음.
- 같은 지역·같은 날이라도 **VIIRS 핫스팟이 훨씬 많다** → 그냥 이어 붙이면 2012년부터 화재가 급증한 것처럼 보인다.
- 이걸 통계적으로 맞추는 게 **하모나이제이션(harmonization)**. 위성 분야에서 흔한 문제라 "sensor intercalibration", "data continuity"로도 불린다.

### 리소스
| 리소스 | 설명 | 링크 |
|---|---|---|
| NASA MODIS 공식 사이트 | 센서 개요 | https://modis.gsfc.nasa.gov/ |
| VIIRS 375 m 화재 제품 (UMD) | VIIRS 화재 제품 정보·User Guide | https://viirsland.gsfc.nasa.gov/ ✅ 또는 modis-fire.umd.edu 내 VIIRS 섹션 |
| MODIS Collection 6.1 Active Fire User Guide | 컬럼, confidence, type 정의 (PDF) | modis-fire.umd.edu → Guides |
| VIIRS 375 m Active Fire User Guide | 동일 (VIIRS) | modis-fire.umd.edu → Guides ✅ |
| 검색어 | "MODIS VIIRS active fire comparison", "MODIS VIIRS fire continuity" | Google Scholar |

---

## 6. 벤치마킹: 비슷한 기존 서비스

만들 앱의 레퍼런스. 이미 "화재 계절 그래프"나 "평년 대비 비교"를 제공하는 곳이 있으니 UI 아이디어를 얻자.

| 서비스 | 볼 포인트 | 링크 |
|---|---|---|
| Global Forest Watch – Fire alerts | 지역 선택 → VIIRS 화재 알림 수의 **주간 그래프 + 과거 평균 범위** 비교 | https://www.globalforestwatch.org/ |
| GWIS (Global Wildfire Information System, EU JRC) | 국가별 화재 통계, **계절 프로필** | https://gwis.jrc.ec.europa.eu/ |
| FIRMS 자체 차트 기능 | FIRMS 지도 내 통계 기능 확인 ✅ | https://firms.modaps.eosdis.nasa.gov/ |
| NASA Earth Observatory 월별 화재 지도 | 계절별 화재 이동 패턴 | §3 링크 |
| GitHub contribution 그래프 / 날씨 "warming stripes" | **캘린더 히트맵 디자인** 참고 | https://showyourstripes.info/ |

---

## 7. 데이터 분석 개념

| 개념 | 쉬운 설명 | 이 프로젝트에서의 용도 |
|---|---|---|
| 시계열 집계 | 날짜별 데이터를 월/연 단위로 묶어 세기 | 캘린더 셀 값 |
| 기후값(Climatology) / 기준기간 | 여러 해 같은 달의 평균/중앙값 = "평년" | "평년 대비" 비교 기준 |
| 중앙값(Median) vs 평균 | 대형 화재 한 해가 평균을 크게 끌어올리므로 중앙값이 더 견고 | 원문도 median 언급 |
| 이상치(Anomaly) | 평년 대비 얼마나 벗어났는지 (%, 백분위, z-score) | "이번 달 +180%" |
| MAD / Robust z-score | 이상치에 강한 표준편차 대용 | 이상치 등급 |
| 그리드 집계(Binning) | 점 데이터를 격자 칸에 담아 세기 | 데이터 용량 축소, 해상도 차이 완화 |
| 선형/로그 회귀 | 두 변수 관계를 직선으로 맞추기 | MODIS → VIIRS 변환식 |
| 분위수 매핑(Quantile mapping) | 한 분포를 다른 분포 모양에 맞추기 | 하모나이즈 대안 |
| 교차검증 | 일부 기간으로 학습, 다른 기간으로 검증 | 보정 정확도 증명 |
| 변화점 탐지 | 시계열이 갑자기 꺾이는 지점 찾기 | 2012년 단절이 사라졌는지 확인 |

### 리소스
| 리소스 | 링크 |
|---|---|
| pandas 시계열 (resample, groupby) | https://pandas.pydata.org/docs/user_guide/timeseries.html |
| Kaggle Learn (pandas, 데이터 시각화 무료 강의) | https://www.kaggle.com/learn |
| 분위수 매핑 개념 검색어 | "quantile mapping bias correction" |
| scipy.stats / statsmodels (회귀, 통계) | https://www.statsmodels.org/ |

---

## 8. 기술 스택 튜토리얼 (역할별)

### 데이터 처리 (Python)
| 도구 | 용도 | 링크 |
|---|---|---|
| pandas / polars | CSV 처리, 집계 | https://pandas.pydata.org/ · https://pola.rs/ |
| DuckDB | 수천만 행 CSV/Parquet을 SQL로 빠르게 (강력 추천) | https://duckdb.org/docs/ |
| GeoPandas / Shapely | 폴리곤 안의 점 필터링 | https://geopandas.org/ |
| Jupyter | 탐색 분석 | https://jupyter.org/ |

### 백엔드
| 도구 | 용도 | 링크 |
|---|---|---|
| FastAPI | Python API 서버 | https://fastapi.tiangolo.com/ |

### 프론트엔드 (지도·차트)
| 도구 | 용도 | 링크 |
|---|---|---|
| Leaflet + Leaflet.draw | 쉬운 지도 + 영역 그리기 | https://leafletjs.com/ |
| MapLibre GL JS | 고성능 벡터 지도 (대안) | https://maplibre.org/ |
| Apache ECharts | 캘린더 히트맵/차트 (heatmap 예제 풍부) | https://echarts.apache.org/examples/ |
| Plotly | 빠른 인터랙티브 차트 | https://plotly.com/javascript/ |
| DuckDB-WASM | 브라우저에서 Parquet 쿼리 (서버 없이 배포) | https://duckdb.org/docs/api/wasm/overview ✅ |

### 지리 데이터 기초 개념
- **WGS84 (EPSG:4326)**: 일반 위경도 좌표계. FIRMS 좌표가 이것.
- **Bounding box (bbox)**: `서, 남, 동, 북` 경계 사각형. FIRMS API 영역 지정 형식.
- **GeoJSON**: 폴리곤/점 데이터를 JSON으로 표현하는 표준. 프론트엔드에서 그린 영역을 백엔드로 보낼 때 사용.
- 리소스: https://geojson.io/ (폴리곤 그려보며 GeoJSON 구조 익히기)

### 접근성 있는 색상
| 리소스 | 링크 |
|---|---|
| ColorBrewer (YlOrRd, RdBu 등 검증된 팔레트) | https://colorbrewer2.org/ |
| Viridis 팔레트 소개 | https://cran.r-project.org/web/packages/viridis/vignettes/intro-to-viridis.html |
| 색각이상 시뮬레이터 (Coblis) | https://www.color-blindness.com/coblis-color-blindness-simulator/ |

---

## 9. (선택) 논문 — 깊게 이해하고 싶을 때

발표에서 "근거 있는 방법론"을 보여줄 때 인용하기 좋다. Google Scholar에서 제목으로 검색.

| 논문 | 왜 읽나 |
|---|---|
| Giglio, Schroeder & Justice (2016), *The collection 6 MODIS active fire detection algorithm and fire products*, Remote Sensing of Environment | MODIS 화재 탐지 원리 표준 문헌 |
| Schroeder et al. (2014), *The New VIIRS 375 m active fire detection data product: Algorithm description and initial assessment*, Remote Sensing of Environment | VIIRS 375 m 제품 원리, MODIS와의 비교 |
| 검색어: "MODIS VIIRS active fire continuity", "fire seasonality satellite climatology" | 하모나이즈·화재 계절성 선행 연구 |

> 처음엔 Abstract와 그림만 보고, MODIS 대비 VIIRS 탐지량 차이를 다룬 부분만 찾아 읽어도 충분하다.

---

## 10. 해커톤 운영 관련

| 리소스 | 링크 |
|---|---|
| 챌린지 페이지 (Resources 탭 11/13 공개) | https://www.spaceappschallenge.org/2026/challenges/modis-and-viirs-hot-spots/ |
| Space Apps 공통 리소스 / 참가 가이드 | https://www.spaceappschallenge.org/ 상단 Resources 메뉴 ✅ |
| 과거 우승작 둘러보기 (화재 관련 "Spot That Fire" 2018–2020) | 예: https://2019.spaceappschallenge.org/challenges/living-our-world/spot-fire-v20/ |

---

## 11. 첫날 체크리스트 (입문자용)

- [ ] §2 Earthdata 원격탐사 기초 글 읽기
- [ ] FIRMS 지도에서 MODIS/VIIRS 레이어 켜고 끄며 차이 눈으로 확인
- [ ] Earthdata 계정 + FIRMS MAP_KEY 발급
- [ ] 국가별 CSV 하나 받아서 컬럼 의미 파악 (§4, `02_data_and_harmonization.md` 컬럼 표 참고)
- [ ] Global Forest Watch에서 지역 하나 골라 화재 그래프 구경
- [ ] `01_challenge_overview.md` 다시 읽고 이해 안 되는 용어 표시해두기
