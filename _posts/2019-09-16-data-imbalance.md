---
layout: post
title:  "Data imbalance summary"
date:   2019-09-16 12:46:00
tags: [공부, data science, data imbalance, SMOTE, sampling]
comments: true
---

Data imbalace 문제에 대한 설명과 해결방법에 대한 요약입니다. 기록용으로 작성되었습니다.

## Dataset imbalance problem

- 한 클래스가 다른 클래스들 보다 매우 많은 샘플 데이터를 포함하고 있는 데이터 셋
- 적어도 한 클래스가 매우 적의 수의 샘플 데이터만 포함한 경우 (minority class)
- 일반적으로 imbalance 데이터에 대해 분류기가 majority class에서는 좋은 정확도를 보이지만, minority class들에서는 낮은 정확도를 보임

### Ex) 신용카드 fraud detection

- 신용카드 이상거래의 경우 정상 거래 케이스에 비해 케이스가 매우 적음
- 정상거래가 99%, 이상 거래가 1% 인 경우, 모든 케이스를 정상거래로 분류하기만 해도 전체 99%의 정확도를 얻을 수 있음.
- 그러나 이상거래에 대한 정확도만 보면 매우 낮음.

## Solution
### 1. Sampling

- class imbalance를 낮추기 위해 training 데이터를 resampling하는 방법
- test data에는 적용하지 않음!

#### 1) Under/Down sampling

- majority class의 subset을 사용하는 방법.
- 장점: 데이터가 balance되고 학습속도도 더 빨라짐
- ex) RUS (random majority under-sampling)
- 단점: 중요한 학습 데이터를 없앨 수 있는 위험이 있음

#### 2) Oversampling

- minority class를 중복해서 사용함으로써 class imbalance를 해소하는 방법
- 장점: 정보의 손실이 없음, under-sampling보다 성능이 좋음
- 단점: 데이터를 복제하기 때문에 overfitting의 위험이 있음

#### 3) SMOTE(Synthetic Minority Over-sampling Technique)

- minor class의 분포와 일관성있는 새로운 샘플을 만들어내는 방식
- 고차원 데이터에는 효과적이지 않음

![](https://s3-ap-south-1.amazonaws.com/av-blog-media/wp-content/uploads/2017/03/16142852/ICP3.png)


### 2. Ensemble

데이터를 sampling 하는 방법 외 알고리즘 적으로는 주로 ensemble method를 적용하여 성능을 개선할 수 있음. (내용 생략)

## Reference

- [https://towardsdatascience.com/comparing-different-classification-machine-learning-models-for-an-imbalanced-dataset-fdae1af3677f](https://towardsdatascience.com/comparing-different-classification-machine-learning-models-for-an-imbalanced-dataset-fdae1af3677f)
- [https://www.analyticsvidhya.com/blog/2017/03/imbalanced-classification-problem/](https://www.analyticsvidhya.com/blog/2017/03/imbalanced-classification-problem/)
