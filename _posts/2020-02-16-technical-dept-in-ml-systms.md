---
layout: post
title:  "Hidden Technical Debt in Machine Learning Systems"
date:   2020-02-16 16:03:00
tags: [공부, feature store, machine learning, mlops]
comments: true
---

ML 서비스를 Production 에 적용하기 시작하면서, CD4ML, ML Ops 에도 관심이 많이 생겨서 차근차근 공부해보려고 합니다.

나온지 꽤 되긴 했는데 ML system 에서의 기술 부채에 대해 Google 에서 작성한 유명한 논문인 [Hidden Technical Debt in Machine Learning Systems (2015)](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) 찬찬히 읽어보고 요약했습니다.

## 1. Introduction - ML 시스템에서의 기술적 부채

["Technical Debt" (기술 부채)]((https://en.wikipedia.org/wiki/Technical_debt)) 란?

Ward Cunningham 이 제시한 metaphor로, 대략 다음의 한 짤로 요약됩니다...

![](https://miro.medium.com/max/863/1*55Jn564sqMdKT39XIW3uxQ.png)

~~언제까지 네모난 바퀴로 갈텐가...~~

소프트웨어 엔지니어링에서와 마찬가지로 라이브 시스템에 적용되는 ML 시스템에서도 `개발과 배포에 걸리는 시간은 상대적으로 빠르고 저렴하지만, 이를 유지보수하는 것은 어렵고 비용이 많이 들게되는` 현상이 광범위하게 발생하고 있습니다.

특히 ML system 에서는 `전통적인 코드를 유지 보수하는 문제에 추가로 ML에 특정된 이슈들로 인한 기술적 부채들도 발생`합니다. 그런데 이런 기술 부채들은 코드 레벨보다는 시스템 레벨에 존재하기 때문에 감지하기 어렵습니다. ML 시스템에서 코드 외 `시스템에서 흘러가는 데이터` 때문에 전통적인 추상화 경계(abstraction boundaries)가 깨지기 쉽고, 코드 레벨의 기술부채를 갚는 전형적인 방법들 (e.g. 리팩토링, 테스트코드 작성 등)로는 ML system의 부채를 해결하는데 충분하지 않습니다.

이 논문에서는 ML 기술 부채가 빠르게 축적될 수 있는 ML system 내의 시스템 레벨의 상호작용과 인터페이스에 집중하여 ML 기술 부채를 발생시키는 요소들에 대해 제시하고 있습니다.

## 2. Complex Models Erode Boundaries

전통적인 소프트웨어 엔지니어닝에서는 캡슐화와 모듈 디자인를 사용한 강한 추상화 경계(abstraction boundaries)를 통해 유지보수가 가능한 코드를 만들고, 독립된 작은 변화와 개선을 만들기 쉽게 합니다. 엄격한 추상화 경계는 인풋과 아웃품에 대한 일관성을 가지게 합니다.

그러나 ML 시스템에서는 `설계했던 동작이 외부의 데이터 의존성을 제외하고 소프트웨어 로직만으로는 효과적으로 표현 될 수 없기 때문에` 엄격한 추상화 경계를 적용하기 어렵고, 이로 인해 심각한 기술 부채를 만들 수 있습니다.

### Entanglement

ML 시스템에서는 신호들을 섞고 얽기 때문에, 독립적인 개선을 하기 어렵습니다. 예를 들어 x_1, ... , x_n 의 feature를 사용하는 모델에서 만약 x_1 의 입력 분포를 바꾸는 경우, 나머지 모든 n-1개의 feature 들의 가중치, 사용 여부 등이 모두 바뀌게 됩니다. 새로운 n+1 번째 feature 를 추가하거나, 특정 feature x_j 를 제거하는 경우도 마찬가지 입니다.

이 논문에서는 이러한 경우를 CASE (`Changing Anything Changes Everything`) 원칙으로 명명했습니다. CASE 원칙은 입력값에 대해서 뿐아니라, 하이퍼파라미터, 학습에 필요한 설정들, 샘플링 방법등 ML system 의 모든 가능한 작업에 적용될 수 있습니다.

(해결책 TBD)

### Correction Cascades

문제 A 를 풀기 위한 모델 m_a 가 존재할 때, 문제 A와 약간만 다른 문제 A'에 대한 해결책이 필요할 때, 빠른 해결책으로 모델 m_a를 입력값으로 하고 일부분만 재학습해서 사용하는 m_a'를 만들 수 있습니다.

그러나, 이렇게 만들어진 모델의 경우 모델 m_a에 대한 의존성을 가지고 있어 차후에 모델을 개선하기 위해 분석하는 비용이 많이 들게됩니다.

(해결책 TBD)

### Undeclared Consumers (SE에서도 동일하게 발생하는 문제 같은데)

모델 m_a를 통한 예측 결과가 다른 시스템에서 널리 접근가능하게 만들어진 경우, 접근 제한이 없는 경우 선언하지 않은 소비자들이 모델의 출력값을 사용하게 되는 경우가 발생합니다.

## 3. Data Dependencies Cost More than Code Dependencies

전통적인 소프트웨어 엔지니어링에서 `Dependency dept`가 코드를 복잡하게 하고 기술 부채를 발생시키는 키 요소로 지목되었는데, ML 시스템에서도 이와 비슷하게 `data dependency` 가 기술 부채를 쌓고 있으며, 더 감지하기도 어렵다는 사실을 발견했습니다. 코드 의존성의 경우 compiler, linker 등 static analysis 를 통해 확인할 수 있습니다. 데이터 의존성의 경우도 의존성을 파악할 수 있는 도구가 없다면, 대규모의 해결하기 어려운 데이터 의존성을 쌓기 쉽습니다.

### Unstable Data Dependencies

모델에서 다른 시스템에서 만든 출력값을 모델의 입력값으로 사용하게 되는 경우가 많은데, 이런 경우 시간의 흐름에 따라 질적으로, 양적으로 변화하는 unstable 한 입력값이 될 수 있습니다. 예를 들어 다른 모델의 출력 값을 입력값으로 사용하는 경우 해당 모델이 변경됨에 따라 출력값이 암묵적으로 바뀔 수 있습니다. 이러한 입력값의 변화는 이를 사용하는 시스템에서 감지하기 어렵기 때문에 위험합니다.

이에 대한 한 가지 해결 전략으로는 데이터에 대해 versioned copy 를 만들어 데이터 의존성을 해결하는 방법이 있습니다.

- 논문에는 없지만, 지난번에 사용해보고 작성한 데이터 버전 관리 툴 [DVC](https://inahjeon.github.io/dvc/) 글 참고

### Underutilized Data Dependencies

활용도가 낮은 데이터들.

- Legacy Features
- Bundled Features
- ϵ-Features
- Correlated Featuress

### Static Analysis of Data Dependencies

전통적인 코드에서는 컴파일러와 빌드 시스템에서 의존성 그래프에 대해 정적 검사를 수행할 수 있습니다. 데이터 의존성에 대한 정적 분석 도구는 비교적으로 적고, 자동화된 feature 관리 같은 도구를 예를 들 수 있습니다.

## 4. Feedback Loops

Live ML 시스템에서 한 가지 중요한 특성은 시간이 흐름에 따라 그들의 행동이 모델에 영향을 끼치게 되는 것입니다. 이러한 특성은 모델이 배포되기 전에는 시스템이 어떻게 동작할 지 예측하기 어려운 analysys debt 를 발생시킬 수 있습니다.

(모델이 데이터에 영향을 끼치고, 다시 그 데이터가 모델에 영향을 끼치는 건가 봉가)

### Direct Feedback Loops

### Hidden Feedback Loops

## 5. ML-System Anti-Patterns

![](https://user-images.githubusercontent.com/16538186/74599412-be9d1880-50c4-11ea-919b-8d0971757405.png)

Real-world ML system 에서 실제 학습이나 예측에 사용되는 "ML 코드"가 차지하는 비중은 매우 적고, ML system 을 위한 요구되는 인프라가 매우 복잡합니다.

아래에서는 ML 시스템에서 반드시 피하거나, 리팩토링 되어야하는 몇 가지 system-design anti-pattern 들에 대해 다룹니다.

### Glue Code.

### Pipeline Jungles.

### Dead Experimental Codepaths

### Abstraction Debt

### Common Smells.

- Plain-Old-Data Type Smell.
- Multiple-Language Smell.
- Prototype Smell.

## 6. Configuration Debt

## 7. Dealing with Changes in the External World

### Fixed Thresholds in Dynamic Systems.

### Monitoring and Testing.

## 8. Other Areas of ML-related Debt

### Data Testing Debt.

### Reproducibility Debt.

### Process Management Debt.

### Cultural Debt.

## 9. Conclusions: Measuring Debt and Paying it Off


