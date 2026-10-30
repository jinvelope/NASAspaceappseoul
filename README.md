# NASA Space Apps Challenge 2026 — Seoul

**챌린지: [Harmonization of MODIS and VIIRS Hot Spots](https://www.spaceappschallenge.org/2026/challenges/modis-and-viirs-hot-spots/)**

## 프로젝트 소개

위성은 20년 넘게 지구의 화재를 관측해 왔지만, 그 기록은 두 종류의 센서로 나뉘어 있습니다.

- **MODIS** (Terra/Aqua, 2000년~): 기록은 길지만 해상도가 1 km이고 수명이 끝나가는 중
- **VIIRS** (S-NPP/NOAA-20/NOAA-21, 2012년~): 해상도 375 m로 더 많고 작은 불까지 탐지

두 센서는 해상도·민감도·관측 시각이 달라서 기록을 그대로 이어 붙이면 비교할 수 없습니다.
이 프로젝트의 목표는 두 기록을 **센서와 무관한 하나의 시계열로 통합(harmonize)** 하고,
사용자가 지도에서 고른 지역의 **연소 활동 캘린더(월 × 연도 화재 히트맵)** 를 보여주는 웹 앱을 만드는 것입니다.

### 주요 기능 (계획)

- 지도에서 관심 지역 그리기 또는 위경도 입력
- 2000년대 초 ~ 현재까지의 월별 화재 캘린더 (색각이상을 고려한 색상)
- 이번 달 화재 활동을 과거 중앙값과 비교 (예: "평년 대비 +180%")
- 핫스팟 원본 및 집계 결과 CSV 다운로드

## 데이터

- [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/) — MODIS·VIIRS 화재 핫스팟

## 저장소 구조

```
.
├── README.md        # 이 문서
└── 0.common/        # 팀 공통 문서 (챌린지 분석, 데이터, 기획, 학습 자료)
```

## 문서

| 문서 | 내용 |
|---|---|
| [챌린지 개요](0.common/01_challenge_overview.md) | 원문 요약, 요구사항, 산출물 체크리스트 |
| [데이터 & 통합 기법](0.common/02_data_and_harmonization.md) | MODIS/VIIRS 비교, FIRMS 데이터, 통합 기법 후보 |
| [솔루션 기획안](0.common/03_solution_plan.md) | 아키텍처, 기능 우선순위, 역할·일정, 리스크 |
| [학습 리소스](0.common/04_learning_resources.md) | 입문자용 학습 순서와 링크 모음 |

처음 합류했다면 **챌린지 개요 → 학습 리소스** 순서로 읽어주세요.

## 일정

- 2026-11-13 (금): 챌린지 공식 Resources 공개
- 해커톤 시작 전에는 조사·기획만 진행합니다 (대회 규정: *"Do not begin work on your projects until the hackathon begins."*)
