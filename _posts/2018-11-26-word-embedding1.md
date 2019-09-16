---
layout: post
title:  "Word Embeddings 1"
date:   2018-11-26 01:45:00
tags: [공부, data science, nlp]
comments: true
---

# An Intuitive Understanding of Word Embeddings: From Count Vectors to Word2Vec - 번역 (1)

개인 공부 용 번역글 입니다.

Reference: 

[An Intuitive Understanding of Word Embeddings: From Count Vectors to Word2Vec](https://www.analyticsvidhya.com/blog/2017/06/word-embeddings-count-word2veec/)

# Word Embedding 이란

---

**Word embedding**은 대부분의 기계 학습 알고리즘에서 원시 문자열이나 텍스트 정보를 처리할 수 없기 때문에, 정보를 처리하고 학습 알고리즘에 이용하기 위해 **숫자 Vector형태로 단어를 변환**하는 것이다. 

가장 간단한 방법으로는 **'고유 단어 목록' 사전을 이용해서 one-hot encoding**을 통해 vector로 변환 할 수 있다.

예를 들어 다음과 같은 고유 단어 사전이 있을 때,

['사과', '언어', '커피', '영화'] 

'사과' 라는 단어를 [1, 0, 0, 0] 이라는 숫자 vector로 변환할 수 있다.

# Word Embedding의 유형

---

1. Frequency based Embedding
2. Prediction based Embedding

# Frequency based: Count Vector

---

D개의 문서 {d1, d2, ..., dD} 와 N개의 고유한 단어 토큰을 가진 corpus C가 있다고 생각해보자.

Count vecor 행렬 M은 D X N의 크기를 가지고, 각 행은 문서 D(i)에서 각 단어 토큰의 frequency를 나타낸다.

예를 들어,

D1: He is a lazy boy. She is also lazy.

D2: Neeraj is a lazy person.

이라는 두 문장이 주어졌을 때,

고유 단어 토큰은 [He, She, lazy, boy, Neeraj,  person] 로 구성된다.

그러면 count matrix M은 다음과 같은 2 x 6 행렬로 표현할 수 있다.

 | He  |  She  |  lazy  |  boy |  Neeraj  |  person
-|-|-|-|-|-
D1 | 1   |     1   |      2     |    1    |      0        |     0
D2 | 0     |   0     |   1    |     0      |    1    |         1

matrix M 의 행은 문서들, 열은 단어 토큰에 대응된다.

Count vector는 여러 변형이 있는데, 일반적으로 다음과 같이 나누어진다.

1) dictionary를 구성하는 방법

실세계 언어는 수많은 단어로 구성되어 있기 때문에 모든 단어를 다 사용하면, 매우 sparse한 matrix가 되고, 쓸모없이 크기 때문에 계산하는 데에도 매우 비효율적이다. 그렇기 때문에 대안으로 10,000개의 상위 빈도를 가지는 단어들만으로 사전을 구성해서 사용하기도 한다.

2) 각 단어를 세는 방법

각 단어가 문서에 얼마나 나타났는지 빈도(**frequency**)를 세는 방법과 단순히 나타났는지 아닌지(**presence**) 여부로 표현 할 수 있다. 일반적으로 빈도를 세는 방법이 더 많이 사용된다.

# Frequency based: TF-IDF Vector

---

Count vector와 동일하게 빈도 기반의 벡터화 방법이지만, 단일 문서 내에서의 단어 출현 빈도 뿐 아니라 전체 코퍼스에서의 출현 빈도를 고려한 방법.

'is', 'the', 'a' 등과 같은 일반적인 단어는 거의 모든 문서에서 높은 빈도로 나타나기 때문에, 거의 모든 문서에서 발생하는 일반적인 단어의 비중을 줄이고, 각 문서에서 특별하게 많이 나타나는 단어의 가중치를 높게 주도록 동작하는 방법.

TF-IDF 계산방법

예)

Document1: (This, 1), (is, 1), (abount , 2), (Messi, 4)

Document2: (This, 1), (is, 2), (abount , 1), (Tf-idf, 1)

로 주어진 코퍼스가 있을 때,

TF(Term Frequency, 단어 빈도) = (용어 t가 문서에 나타나는 횟수) / (문서의 용어 수)

-----------
TF(This, Document1) = 1/8

TF(This, Document2) = 1/5

TF(Messi, Document2) = 4/8

-----------

IDF(Inverse Document Frequency, 역문서 빈도) = log(N/n) (N: 문서의 수, n은 용어 t가 출현한 문서의 수)

-----------
IDF(This) = log(2/2) = 0

IDF(Messi) = log(2/1) = 0.301

TF-IDF (This, Document1) = (1/8) * (0) = 0

TF-IDF (This, Document2) = (1/5) * (0) = 0

TF-IDF (Messi, Document1) = (4/8) * (0.301) = 0.15

-----------

# Frequency based: Co-Occurrence Vector

---

유사한 단어가 함께 발생하는 경향이 있고, 비슷한 문맥을 갖는 다는 점에서 아이디어를 얻어, 단어들의 동시 발생 횟수로 행렬을 구성하는 방법

Co-occurrence - 주어진 코퍼스에 대해 단어 쌍의 co-occurrence는 단어 w1, w2가 context window에 함께 나타난 횟수를 의미함.

Context window - co-occurrence를 측정하기 위한 범위를 의미하는 window

-----------

ex) 다음과 같은 문장에서 context window size = 2로 주어졌을 때,

[Quick	Brown	**Fox**	Jump	Over]	The	Lazy	Dog

 Fox라는 단어와 동시 발생한 단어는 Quick, Brown, Jump, Over이다.

Quick	Brown	[Fox	 Jump	**Over**	The	Lazy]	Dog

비슷하게 Over 와 동시 발생한 단어는 Fox, Jump, The, Lazy가 된다.

-----------

코퍼스의 단어 크기가 V일 때, Co-occurrence 행렬의 크기는 V x V 가 된다. 일반적으로 V x V 는 매우 크기가 크기 때문에, 불용어와 같이 불필요한 단어를 제거하여 사용한다. 그렇지만, 여전이 단어 집합의 크기가 매우 크기 때문에, PCA, SVD등의 기법을 활용하여 차원을 축소하거나, 행렬 인수 분해를 사용한다.



(Word Embedding 2 에서 계속..)
