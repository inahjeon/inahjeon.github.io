---
layout: post
title:  "Ensemble 요약"
date:   2019-09-10 02:06:00
tags: [공부, data science, ensemble, 앙상블, bagging, boosting, stacking]
comments: true
---

Hands-On Machine Learning with Scikit-Learn and TensorFlow 책에서 설명하고 있는 앙상블 기법에 대한 요약입니다. 기록용으로 작성되었습니다.

## Ensemble learning

- 여러 예측기를 연결해서 더 나은 예측기를 만드는 방법.
- 성능이 낮은 분류기라도 여러개를 모았을 때 좋은 성능을 발휘하는 경우가 많음.
- 각 예측기가 가능한 한 서로 독립적일 때 최고의 성능을 발휘함.
- 다양한 예측기를 만드는 방법으로는 각기 다른 알고리즘으로 학습시키거나, 다른 데이터셋으로 훈련하는 방법이 있음.

## Ensemble methods

앙상블 학습을 하는 알고리즘

### 앙상블을 적용한 투표 기반 분류기

- hard voting classifier: 각 분류기의 예측을 모아서 가장 많이 선택된 클래스를 예측하는 방법
- soft voting classifier: 각 분류기에서 클래스 별 확률 값을 구할 수 있는 경우, 클래스 별 확률의 평균값으로 확률이 가장 높은 클래스를 예측하는 방법

### bagging & pasting

원본 데이터 셋을 bagging이나 pasting을 활용하여 여러 개의 샘플 데이터 셋으로 나누고 각각의 모델에 학습 시킨 후, 분류기일 경우 최빈 클래스로 분류, 회귀일 경우 평균값으로 예측함

#### bagging(boostrap aggregating)

- training set 에서 중복을 허용하여 샘플링하는 방식 (중복을 허용하는 리샘플링: boostraping)
- 일반적으로 bagging이 pasting 보다 더 나은 성능을 보임

#### pasting

- 중복을 허용하지 않고 샘플링 하는 방식

#### feature sampling
- random subspaces method: feature sampling 만 사용
- random patches method: training set sampling과 feature sampling을 모두 사용

### boosting

- 약한 학습기를 여러 개 연결하여 강한 학습기를 만드는 앙상블 방법. 앞의 모델을 보완해나가면서 일련의 예측기를 학습시키는 방법.

#### ada boost

- 이전 모델이 under fitting 했던 훈련 샘플의 가중치를 더 높임. 
- 새로운 예측기는 학습하기 어려운 샘플에 점점 더 맞춰지게 됨. 
- 분류기를 연속적으로 학습해야 하기 때문에 병렬로 수행하지 못해 확장성이 떨어짐

> 샘플 → 분류기1 → 가중치샘플 → 분류기2 → .. → 최종 분류기

#### gradient boost

- ada boost처럼 예측기를 순차적으로 추가하는 방식으로, 이전 예측기가 만든 잔여 오차(residual error)에 새로운 예측기를 추가하는 방식

> classifier_n = y - classifier_n-1.predict(x)

> y_pred = sum(classifier.predict(x) for classifier in classifiers)

- stocastic gradient boosting: 훈련 샘플의 비율을 지정해서 각 분류기가 무작위로 샘플링한 훈련셋을 학습하게 함.

### stacking & blending

앙상블에 속한 예측기들의 결과를  취합하는 모델을 학습시키는 방법

- stacking: 훈련셋을 out-of-fold (cross-validation) 방식으로 나누어서 학습시킴
- blending: 훈련셋을 hold-out 방식으로 나누어서 학습시킴

## 참고자료

- Hands-On Machine Learning with Scikit-Learn and TensorFlow
- [https://www.analyticsvidhya.com/blog/2018/06/comprehensive-guide-for-ensemble-models/](https://www.analyticsvidhya.com/blog/2018/06/comprehensive-guide-for-ensemble-models/)
