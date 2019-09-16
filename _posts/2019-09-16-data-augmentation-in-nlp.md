---
layout: post
title:  "Data augmentation in text classification"
date:   2019-09-16 21:40:00
tags: [공부, data science, data augmentation, text, classification, nlp]
comments: true
---

[EDA: Easy Data Augmentation Techniques for Boosting Performance on
Text Classification Tasks](https://arxiv.org/pdf/1901.11196v1.pdf) 이라는 논문을 읽고 요약한 내용입니다.

## 1. Summary:

- NLP에서 범용적으로 적용가능한 Data augmentation technique (EDA: Easy Data Augmentation)을 제안함.
- 제안한 방법 중 Synonym replacement에 대하서만 기존 연구결과가 있음. RI, RS, RD는 논문에서 새롭게 제안한 방법.
- 실험 했던 5개의 benchmark task (classifier - `RNN`, `CNN`)에서 성능 개선이 있었고 특히 적은 샘플 데이터셋에서 효과가 있었음.
- 전체 학습셋 중 50%의 학습셋만 가지고 100% 학습셋을 사용했을 때의 최고 성능과 동일한 정확도를 낼 수 있었음.

## 2. 제안한 방법:

### Text data augmentation operations
- **Synonym Replacement (SR)**: 문장에서 stop word가 아닌 랜덤 n개의 단어를 선택한 후, 랜덤하게 각 단어를 동의어로 변경.
- **Random Insertion (RI)**: 문장 내 특정 랜덤 단어의 랜덤 동의어를 찾아서, 문장 내 랜덤 위치에 동의어를 삽입하는 방식. n회 반복함.
- **Random Swap (RS)**: 문장 내 두개의 단어를 랜덤으로 선택해서 두 단어의 위치를 변경함. n회 반복함.
- **Random Deletion (RD)**: 문장 내 단어를 랜덤하게 p의 확률로 제거함.

### Parameter n, α 설정
- 문장의 길이에 따라 `n`이 변경되도록 문장의 길이 (`l`)을 수식에 반영함.
- `n = αl` (`α`: 문장 내 변경되는 단어의 비율, `l`: 문장의 길이)
- 각 문장마다 `n`개의 augmented 문장을 생성하도록 함.
- RD에서 `p=α`

## 3. 실험방법

- 5개의 benchmark text classification task를 사용
    - SST-2: Stanford Sentiment Treebank ([Socher et al., 2013](https://github.com/AcademiaSinicaNLPLab/sentiment_dataset))
    - CR: customer reviews ([Hu and Liu, 2004; Liu et al., 2015](https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html))
    - SUBJ: subjectivity/objectivity dataset ([Pang and Lee, 2004](http://www.cs.cornell.edu/people/pabo/movie-review-data/)
    - TREC: question type dataset ([Li and Roth, 2002](https://cogcomp.seas.upenn.edu/Data/QA/QC/))
    - PC: Pro-Con dataset ([Ganapathibhotla and Liu, 2008](https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html#datasets))

![](https://user-images.githubusercontent.com/16538186/64959311-d715c680-d8cb-11e9-8d29-a02b0a91f3b9.png)

- 사용한 Classification method
    - LSTM-RNN ([Liu et al., 2016](https://arxiv.org/pdf/1605.05101.pdf))
    - CNN ([Kim, 2014](https://www.aclweb.org/anthology/D14-1181))

## 4. Results

### (1) Performance

- 5개 task에 대해 각 method의 평균 Accuracy를 측정함.

![](https://user-images.githubusercontent.com/16538186/64959312-d715c680-d8cb-11e9-9186-265a0fe8fa33.png)

### (2) Traning set size에 따른 실험 결과

- 사용하는 traning set size를 원본 데이터 크기의 {1, 5, 10, 20, 30, 40, 50, 60, 70, 80, 90, 100} 비율로 늘려가면서 테스트
- 100 사용 시 최고 정확도인 88.3% 정확도를 50%의 데이터를 활용했을 때 얻을 수 있었음 (88.6%)

![](https://user-images.githubusercontent.com/16538186/64959313-d715c680-d8cb-11e9-9ef1-91e0805ad41e.png)

### (3) Conserving true labels

- augmenation으로 만들어진 데이터에 대해 원본 문장의 label이 보존되는지 차원 축소 및 데이터 시각화를 통해 확인함.

- Pro-con data에 대해 `t-SNE` (Van Der Maaten, 2014) 차원축소를 통해 2D plot으로 시각화함.

![](https://user-images.githubusercontent.com/16538186/64959314-d715c680-d8cb-11e9-8025-82e2cb901285.png)

- 생성된 문장의 latent vector가 동일 label을 가진 원본 문장들과 가까운 위치에 있음을 확인할 수 있었음.

### (4) 파라미터 α 에 대한 성능 분석

![](https://user-images.githubusercontent.com/16538186/64959316-d7ae5d00-d8cb-11e9-9177-acca412d1ca9.png)

- SR은 작은 α에 대해 잘 동작했음. 너무 많은 단어를 교체하면 원래 문장과 의미가 많이 달라져서 그럴것으로 해석함.
- RI는 α에 대해 성능이 stable했음. 원래의 문장과 문장순서가 유지되기 때문에 그럴것으로 해석함.
- RS는 α<0.2일때 높은 성능을 보이고 그 이후로는 줄어듦. 문장 내 단어의 순서를 너무 많이 바꾸면 성능이 떨어짐.
- RD는 작은 α에 대해 잘 동작함.
- 대체로 모든 operation들에 대해 α=0.1이 좋은 성능을 보였음.

### (5) How much augmentation? n에 대한 성능 분석

- n={1, 2, 4, 8, 16, 32} 로 실험하여 데이터셋의 크기별 최적의 α, n 값에 대해 제시함.

- 데이터 크기별 추천하는 파라미터값

![](https://user-images.githubusercontent.com/16538186/64959310-d67d3000-d8cb-11e9-8f36-ff609370328a.png)

## 5. Reference
- Paper: [EDA: Easy Data Augmentation Techniques for Boosting Performance on
Text Classification Tasks](https://arxiv.org/pdf/1901.11196v1.pdf)
- Code: [https://github.com/jasonwei20/eda_nlp](https://github.com/jasonwei20/eda_nlp)
