---
layout: post
title:  "Deview 2018 Day2 세션 정리"
date:   2018-10-12 17:14:00
tags: [conference]
comments: true
---

### 1. 인공지능이 인공지능 챗봇을 만든다 (이재원님)
[슬라이드](https://www.slideshare.net/deview/211-119158765)

: 들으면서 리모트 하느라 내용을 거의 못 들음. 아쉽..

### 2. 기계독해 QA: 검색인가, NLP인가? (서민준님)
[슬라이드](https://www.slideshare.net/deview/223-qa-nlp)

QA: Question Answering (궁금한 것이 있을 때 답을 해줄 수 있는 시스템)

1. 검색으로 찾는 QA
("쿼리" 검색 시)
- 내용 및 제목의 관련성 => NLP와 관련!
- 비슷한 검색을 한 유저가 읽은 문서
- 웹 사이트 신뢰도 
등을 고려하여 알맞은 문서를 전달.

(활용하는 기법들)

Word Matching: 검색한 단어가 존재하는 문서를 가져옴 (ctrl-F)
- 제목에만 적용할 경우 꽤 효과적임.
- 쿼리에 단어가 많아질 경우 성능이 좋지 않음.

TF-IDF: 중요 키워드(흔하지 않은 단어)에 더 가중치를 중

LSA: Bag of word
- 각 단어에 추상적인 "태그"를 달아줌
- 추상적인 태그를 통해 다른 단어끼리도 비교할 수 있게 됨.

(검색 != 문장 독해)
- 문장을 읽는 것이 아님. 문법적, 의미적 맥락을 파악못함.
- 원하는 답을 곡 집어서 가져오기 힘듦.

2. NLP로 읽는 QA
: 문서를 시스템이 읽고 중요한 정보를 요약해서 알려주는 형태
- input: 질문, 관련된 문서 (검색 결과)
- output: 답변

답변을 만들때) 

생성모델(Generative model) VS 추출 모델(Extractive model)
- 생성모델의 문제점: 서비스 퀄리티 컨트롤이 어려움, 평가도 어려움

=> 추출 모델을 사용

(7 Milestones in Extractive QA)

Task definition)
- Sentencel-level QA: 문장 레벨의 답변 (관련 문장들 중 알맞은 문장 선택)
- Phrase-level QA: 최소한의 구문으로 답변

* Dataset: SQuAD

Models)
- Cross-attention: 문서를 읽으면서 질문을 참고, 질문을 읽으면서 문서를 참고하는 방식
- Self-attention: 문서를 읽으면서 문서의 다른 부분을 참고
- Transfer learning: Unlabeled corpus를 활용. Wikipedia(3 billion, unlabeled) -> SQuAD(2 million, labeled) 에 적용
- Super-human level: 사람보다 더 높은 성능(91%)을 내는 모델. 앙상블, NLP툴들(파서 등), 적용. 
BERT(93%) - google AI

3. 검색과 NLP의 접점
문제) 문서를 읽는데 너무 오랜 시간이 걸림 (GPU로 6일)

관련 문서 찾기)

Solution1: 검색해서 관련 문서를 찾고 해당 문서를 읽음
- 문제점: 검색엔진을 거쳐야 하고, 검색엔진이 잘못된 답을 내면 성능이 떨어짐.

Solution2: 찾기와 읽기를 동시에 한다.

Q. 검색이 어떻게 문서를 빨리 찾을까, 사람은 정보를 어떻게 빨리 찾을까?

A. 문서들을 미리 범주화 해둠. 
- Locality-Sensitive Hashing(비슷한 아이템의 충돌을 최대화 함) 사용

문서 -> 구문)

문서 d, query q가 주어졌을 때, answer a의 확률을 최대화.
- 기존: 매 새로운 질문마다 F를 재계산 함. F(a, d, q)
- 개선: 따로 나누어서 계산(H는 미리 계산 가능). exp(g(q)) * H(a, d)

...그 뒤는 생략

### 3. Fast & Accurate Data Annotation Pipeline for AI applications (이정권님)
[슬라이드](https://www.slideshare.net/deview/234fast-amp-accurate-data-annotation-pipeline-for-ai-applications-1-119166393)

AI를 이루는 세가지 요소: 하드웨어/모델링/데이터

(어떤 데이터가 필요한가?)
- 늘 쌓이는 데이터: 로그 등 
- Annotation이 필요한 데이터: 영상, 이미지, 음성 등 비정형 데이터 (딥러닝의 주 어플리케이션)

(기존의 annotation과정): 사람이 데이터를 보고 라벨링을 하여 전달 ex) 이미지: 박스를 그리고 라벨링

문제점: 
- 인식의 불일치: 사람마다, 같은 사람이라도 같은 데이터를 보고 다르게 인식할 수 있음.
- 데이터의 부정확함: 사람의 실수로 인해 라벨링이 부정확하거나 빠지는 문제

아이디어:
- 대량생산을 위한 파이프라이닝
- AI로 작업 능률 Boost
- 매 작업과정의 검수

(파이프라이닝)

분업하기
- 사람의 인지단계 고려
- AI의 도움을 받을 수 있도록
- 각 단계별 검증 절차가 용이하게

리콜을 높이자

ex) 주어진 이미지에서 다음 물체들을 모두 찾으세요 VS 다음 영역에 개를 찾으세요.

...생략


### 4. Papago Internals: 모델분석과 응용기술 개발 (신중휘님)
[슬라이드](https://www.slideshare.net/deview/245papago-internals-119175259)

1) Papago MT Engine 팀 역할 소개

(AI 연구/개발과 서비스)
- 모델연구/개발 -> 응용기술 연구/개발 -> 앱/웹 서비스 개발 -> 서버개발
ex) 기계번역 -> 웹 문서 번역 -> 웹문서 -> 최적화

- 집중하는 것: 품질, 편의성, 최적화

2) 실무자에게 듣는 기계번역(seq-to-seq) 모델 분석 방법 - 파파고와 함께: 학습과 수렴 (정권우님)

파파고 기계번역 모델 소개)
- Input: 나는 엄마가 좋아.
- Encoder: input to vector (vectorization)
- Decoder: vector to output (devectorization)
- Ouput: I like mom.

학습과 수렴 관련 연구)

목표: 품질 안정성 (신뢰할 수 있는 학습 모델)
- 학습 / 검증 / 테스트 데이터로 분리
- 매 epoch마다 검증 데이터 자동 평가
- 개선 폭이 적어지는 지점에서 stop (수렴)

기계번역 학습 수렴 기준)

- 정확한 수렴 지점 선별 기준을 명시하지 않음.
- learning rate 기반의 수렴 기준 선택

문제1) 일반화된 모델 수렴 지점 선택 방법이 없음 (=모델 학습이 어려움)
* 일반적인 학습 모델 평가 방법: BLEU
- 학습 과정에서 얼마나 모델이 예민하게 반응하는지 확인 (매 iteration마다 BLEU 점수 측정)

문제2) BLEU != 품질
* 좋은 품질의 번역이란) 
- 동일한 의미 (Adequacy)
- 자연스러운 문장 (Fluency)

: 전문가 평가 점수로 비교해봤을 때 BLEU점수와 품질 점수의 상관관계가 낮음.

3) 실무자에 듣는 기계번역 응용 기술의 연구/개발 - 인공신경망을 활용한 웹사이트 번역 (김재명님)

...생략


### 5. NAVER 광고 deep click prediction: 모델링부터 서빙까지: 김윤중님
[슬라이드](https://www.slideshare.net/deview/226naveraddeepclickprediction)

광고시스템: 광고자 / 광고 / 소비자

미션: 어떤 사용자에게 어떤 광고를 제공할 것인가?

(user, context) -> ad

좋은 광고란? 
- 얼마나 많이 클릭하는가 ex) CTR = clicks/impressions

(Click Prediction Problem)

Dataset:

(u1, c1, a1, / clicked)

(u2, c1, a1, / not clicked)

..

이진 분류문제로 치환

광고문제의 특성)

- 수많은 사용자들, 수많은 형태의 광고
- 높은 성능 -> 매출
- 광고주, 소비자, 광고플랫폼의 utility가 다 다름

목표: (user, context) -> ad

(Naive Approach) Classifier 사용

문제점: 확장성 문제, cold start problem

확장성 문제해결을 위해 case study)
- Taboola, Youtube 등) candidate 추출 (narrow down) -> ranking
- google play) 질의어, candidate -> ranking

(Modeling)

Key idea)
1. Candidate model
2. Embedding (feature space가 매우 sparse함)

User, Context, Ad 모두 Embedidng 한 후, Classification 함

Candidate model 문제: user, context => candidate Ads

preference, similarity를 loss function에 넣음

(user, context), (ad)의 similarity

nearest neighbor문제로 품

(Serving)
- online serving에서 classifier prediction 만 수행
Encoder / Classifier 분리
- Prediction 과정 operation 감축
- Prediction input size줄이기

Common Request Trick)

데이터 Prediction 시 duplicated data는 그룹으로 묶어서 prediction

(성능평가)

similarity로 candidate을 뽑았는데, 100, 500, 1000개로 테스트했을 때 CTR이 1.5배 차이남

-> similarity가 잘 동작하지 않았음.

원인파악: 두루두루 모든 사용자들에게 인기있는 광고는 유사도가 높게 나오지 않았음.
