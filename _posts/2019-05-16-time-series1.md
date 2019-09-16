---
layout: post
title:  "Time Series Analysis & Forecasting (1)"
date:   2019-05-16 02:45:00
tags: [공부, data science, timeseries, forecasting, seasonality, trend, 시계열]
comments: true
---

## Time Series?

Time series (시계열) 데이터는 연도별(annual), 분기별(quarterly), 월별(monthly), 일별(daily) 또는 시간별(hourly) 등 시간의 경과(흐름)에 따라 순서대로(ordered in time) 관측되는 데이터입니다.

### Time series 데이터의 예시:

- 국민 총 생산액, 물가지수, 주가지수 등과 같이 경제활동과 관련된 시계열 (economic time series)
- 일일 강수량, 기온, 연간 지진의 발생 수 등과 같이 물리적 현상과 관련된 시계열 (physical time series)
- 상품판매량, 상품재고량, 상품매출액 등 회사의 경영활동과 관련된 시계열 (marketing time series)
- 총인구, 농가 수, 인구증가율, 평균결혼연령 등과 같이 인구와 관련된 시계열 (demographic time series)
- 품질관리 등과 같은 생산관리와 관련된 시계열(time series in process control)
- 확률과정, 음성파와 같이 통신 공학 또는 공학과 관련된 시계열
- 월별 교통사고 건수, 월별 범죄 발생 수와 같이 사회생활과 관련된 시계열

연속적으로 생성되는 연속시계열(continuous time series)과 이산적 시점에서 생성되는 이산시계열(discrete time series)로 구분할 수 있습니다. 많은 시계열들이 연속적으로 생성되고 있지만 일정한 시차를 두고 관측되므로 이산시계열의 형태를 지니는 경우가 많습니다.

시계열자료(time series data)들은 시간의 경과에 따라 관측된 자료이므로 시간에 영향을 받습니다. 시계열자료를 분석할 때 관측 시점들 간의 시차(time lag)가 중요한 역할을 합니다.
예를 들어, 오늘의 주가가 한달 전, 일주일 전의 주가보다는 어제의 주가에 더 많은 영향을 받는 것과 마찬가지로 가까운 관측 시점일수록 관측 자료들 간에 상관관계가 커집니다.

## Time Series Analysis의 목적

### Modeling

- 시계열자료가 생성된 시스템 또는 확률과정을 모형화하여 과정을 이해하고 제어(control)합니다.
- 예) 여름기간 동안의 시간 당 전기 수요(ED, electricity demand)에 대해 모형화하고 변동 요소(현재 기온, 인구, 시간, 요일 등)들을 제어하여 전기 수요량에 대해 시뮬레이션

### Forecasting

- 과거 시계열자료의 패턴(pattern)이 미래에도 지속적으로 유지된다는 가정하에, 시계열 데이터 모델링을 통해 미래 시점을 예측(Forecasting)합니다.
- 예) 어떤 상품의 매출액 자료를 분석하여 미래의 매출액을 예측


## Components of Time Series

![](https://user-images.githubusercontent.com/16538186/61558650-9d2e6c80-aaa2-11e9-957b-d9bf12eee010.png)

- Trend (추세)
- Cycle (주기성)
- Seasonality (계절성)
- Random Variation (불규칙 변동)

### Trend

- 장기간에 걸쳐 지속적으로 증가 또는 감소하거나 일정한 상태를 유지하려는 패턴입니다.
- Trend를 이용한 데이터 추정: 시간 t에 대한 추세 함수 f(t)와 정상 확률 과정 x(t)로 표현될 수 있습니다.

![](https://user-images.githubusercontent.com/16538186/61558716-bf27ef00-aaa2-11e9-8ad6-286228ac2253.png)

### Cycle

- 시간의 경과(흐름)에 따라 고정된 빈도가 아닌 형태로 상하로 반복되는 변동하는 패턴입니다.
- 추세변동은 장기적으로(일반적으로 1년 초과) 나타나는 추세경향이지만, 순환변동은 대체로 2년 이상의 기간을 주기로 순환적으로 나타납니다. (ex. 경기순환)

![](https://user-images.githubusercontent.com/16538186/61558746-d4048280-aaa2-11e9-9899-e9b30c5612d3.png)

### Seasonality

- 해마다 어떤 특정한 때나 1주일마다 특정 요일 등의 계절성 요인이 시계열에 영향을 줄 때 나타나는 패턴입니다.
- 계절성은 항상 알려진 상수 빈도 형태입니다.

![](https://user-images.githubusercontent.com/16538186/61558779-f4ccd800-aaa2-11e9-8cca-b55578eb672e.png)

### Random Variation

- 시간에 따른 규칙적인 움직임과는 무관하게 랜덤한 원인에 의한 변동성분입니다.

![](https://user-images.githubusercontent.com/16538186/61558818-1037e300-aaa3-11e9-92c5-41814e3c0012.png)


## Stationary vs Non-Stationary

### Stationary
- 시계열 데이터의 통계적 특성(mean, variable,..)이 시간의 흐름에 따라 변하지 않을 때 stationary 라고 말합니다.

### Non-Stationary
- 시계열 데이터의 통계적 특성(mean, variable,..)이 시간의 흐름에 따라 변할 때를 말합니다.
- 많은 시계열 모델링 기법들에서 데이터가 stationary라고 가정하기 때문에, 모델링에 앞서 Non-Stationary 한 데이터를 Stationary 한 데이터로 변형합니다.

## Non-Stationary to Stationary

### Differencing (차분)
- 주로 trend, seasonality를 가지는 시계열 데이터의 경우 활용합니다.
- Differencing: y′t=yt−yt−1.
- Seasonality differencing: y′t=yt−yt−m (m: seasonality 의 주기)
- 언제 적용해야하는가? Dicker-Fuller Test로 검증

![](https://user-images.githubusercontent.com/16538186/61558838-1e85ff00-aaa3-11e9-998a-d62efa2c09c9.png)

### Transformation
- 주로 분산이 커지는 경우 Log 변환을 취하여 안정화 시킬 수 있습니다.
- 분산이 줄어드는 경우는 특정 점으로 수렴함을 의미하므로 모델링이 불필요합니다.

![](https://user-images.githubusercontent.com/16538186/61558875-3a89a080-aaa3-11e9-99d2-a7fd1df54e29.png)

### 전체 변형 적용 예시

![](https://user-images.githubusercontent.com/16538186/61558896-4b3a1680-aaa3-11e9-8073-7e5b8c395ce1.png)
