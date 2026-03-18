<div align="center">

# Jeju Wind Turbine Location Selection

**AHP 방법론을 활용한<br>제주도 풍력발전단지 잠재입지 도출**

**조영서, 김현지, 정민영, 최우영**

2024 한국석유공사 ｢데이터 분석 공모전｣

</div>

## Overview

기후변화와 에너지 안보 위기가 심화되는 가운데, 제주도는 연평균 풍속 4~6m/s의 안정적인 바람 자원을 보유하며 국내 풍력 발전 설비의 약 50%가 집중된 최적지이다.

본 프로젝트는 제주도 기존 풍력발전소 데이터를 기반으로 발전 효율에 영향을 미치는 주요 피처(풍속, 풍향 일관성, 고도, 경사도, 도로 접근성)를 선정하고, 상관관계 분석을 통해 **AHP(Analytic Hierarchy Process)** 로 가중치를 산정하였다. 이를 제주도 전역의 100m 단위 격자에 적용해 신규 풍력발전단지 입지 후보지를 시각화하였다.

---

## Methodology

### 1. 상관관계 분석

기존 풍력발전소의 발전량과 주요 피처 간 상관계수를 산출하여 입지 선정 기준의 우선순위를 도출하였다.

<div align="center">

| 피처 | 상관계수 | 방향 |
| :--: | :------: | :--: |
| 풍향 일관성 | +0.147 | 양 |
| 풍속 | +0.131 | 양 |
| 경사도 | −0.114 | 음 |
| 고도 | −0.133 | 음 |
| 도로와의 거리 | −0.055 | 음 |

</div>

### 2. AHP 가중치 설정

상관계수를 기반으로 쌍대비교 행렬(Pairwise Comparison Matrix)을 구성하고, 각 피처의 가중치를 산정하였다.

$$\text{score} = (x_1 \cdot w_1) + (x_2 \cdot w_2) + (1-x_3) \cdot w_3 + (1-x_4) \cdot w_4 + (1-x_5) \cdot w_5$$

- $x_1$: 풍속, $x_2$: 풍향 일관성 (값이 클수록 유리 → 정방향)
- $x_3$: 도로와의 거리, $x_4$: 경사도, $x_5$: 고도 (값이 작을수록 유리 → 역방향)

### 3. 100m 그리드 생성

제주도 전역을 WGS84(EPSG:4326) 기준 100m × 100m 격자로 분할하고, 보호구역을 제외한 비보호구역 격자점에 풍속·풍향·고도·경사도·도로거리 데이터를 매핑하였다.

### 4. 잠재입지 시각화

격자별 종합 점수를 산출하여 **score ≥ 0.66** 인 고잠재력 지역을 Folium 인터랙티브 지도로 시각화하였다. 스코어에 따라 Blue(0.4) → Lime(0.65) → Red(0.95) 그라데이션을 적용하고, 기존 운영 발전단지는 번개 아이콘으로 표시하였다.

---

## Results

분석 결과, 고잠재력 지역(score ≥ 0.66)은 **해안가와 중산간 지역**에 군집을 이루며 분포하였다. 기존 발전단지 위치와 높은 공간적 일치도를 보여 모델의 유효성을 실증하였고, 중산간 지역과 서귀포 동부 일부는 기존 발전소가 없음에도 높은 잠재력을 보여 신규 후보지로서의 가능성을 확인하였다.

> 인터랙티브 지도: [output/jeju_wind_turbine_map.html](output/jeju_wind_turbine_map.html)

---

## Project Structure

```
Jeju_WindTurbine_Location_Selection/
├── jeju_wind_turbine.ipynb     # 전체 분석 파이프라인
├── output/
│   └── jeju_wind_turbine_map.html  # Folium 인터랙티브 지도
└── input/
    ├── 풍력발전량(2020.3분기).csv  # 제주 발전량 데이터
    ├── 풍력발전량(2020.4분기).csv
    ├── 풍력발전량(2021.1분기).csv
    ├── 풍력발전량(2021.2분기).csv
    ├── 한국에너지공단_풍력기 위치정보_20221231.csv
    ├── wind.csv                    # 기상청 풍속·풍향 데이터
    ├── 고도.csv                    # Google Elevation API 결과
    ├── 서귀포경사도.csv / 제주경사도.csv  # DEM 기반 경사도
    ├── LaLo_roads_street.csv       # 일반국도 좌표
    ├── grid_points.csv             # 100m 격자 좌표
    ├── jeju_1year.csv              # 병합 최종 데이터
    ├── score.csv                   # 격자별 종합 점수
    ├── 보호구역래스터.tif
    └── DEM/                        # 수치표고모델
```

---

## Getting Started

### 패키지 설치

```bash
pip install pandas numpy scipy scikit-learn geopandas rasterio pyproj shapely folium tqdm chardet requests
```

### API 키 설정

노트북 내 Google Elevation API 호출 셀에 발급받은 키를 입력한다.

```python
google_api_key = "YOUR_GOOGLE_API_KEY"
```

> Google Cloud Console에서 Elevation API를 활성화하고 키를 발급받는다.

### 실행 순서

<div align="center">

| 단계 | 섹션 | 내용 |
| :--: | :--- | :--- |
| 1 | 1.1 발전량 데이터 | 분기별 발전량 데이터 병합 |
| 2 | 1.2 위경도·발전용량 | 발전기 위치 및 용량 매핑 |
| 3 | 1.3 고도 | Google Elevation API로 고도 추출 |
| 4 | 1.4 경사도 | DEM 기반 경사도 매핑 |
| 5 | 1.5 풍속·풍향 | 기상청 ASOS/AWS 데이터 join |
| 6 | 1.6 도로와의 거리 | KDTree 기반 최근접 도로 거리 계산 |
| 7 | 2 | 상관관계 분석 및 우선순위 도출 |
| 8 | 3 | AHP 가중치 산정 |
| 9 | 4 | 100m 그리드 생성 및 점수 계산 |
| 10 | 5 | Folium 지도 시각화 |

</div>

---

## Data Sources

| 데이터 | 출처 |
| :----- | :--- |
| 풍력기 위치정보 | [한국에너지공단](https://www.data.go.kr/data/15085304/fileData.do) |
| 발전 용량·발전기 대수 | [제주에너지공사 주요시설](https://www.jejuenergy.or.kr/index.php/contents/energy/facilities) |
| 풍력발전기 전력 정보 | [제주에너지공사 발전량](https://www.data.go.kr/data/15028181/fileData.do) |
| 풍속·풍향 (ASOS/AWS) | [기상자료개방포털](https://data.kma.go.kr/data/grnd/selectAsosRltmList.do?pgmNo=36) |
| DEM (수치표고모델) | [국토지리정보원](https://www.data.go.kr/data/15059920/fileData.do) |
| 보호지역 벡터 | [한국보호지역 통합DB](http://www.kdpa.kr/#) |
| 일반국도 도로중심선 | [국토교통부](https://www.data.go.kr/data/15122482/fileData.do) |
