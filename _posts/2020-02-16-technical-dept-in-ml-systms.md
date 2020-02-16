---
layout: post
title:  "Hidden Technical Debt in Machine Learning Systems"
date:   2020-02-16 18:47:00
tags: [공부, feature store, machine learning, mlops]
comments: true
---

ML 서비스를 Production 에 적용하기 시작하면서, CD4ML, ML Ops 에도 관심이 많이 생겨서 차근차근 공부해보려고 합니다.

나온지 꽤 되긴 했는데 MLOps 관련 많은 article 들에서 읽어볼 것을 추천하고 있는, [Hidden Technical Debt in Machine Learning Systems (2015): ML system 에서의 기술 부채에 대해 Google 에서 작성한 유명한 논문](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) 를 찬찬히 읽어보고 적당히 의역 및 요약해보았습니다. (정확한 내용은 원문을 참고해주세요.)

## 1. Introduction - ML 시스템에서의 기술적 부채

### [Technical Debt (기술 부채)](https://en.wikipedia.org/wiki/Technical_debt) 란? 

Ward Cunningham 이 제시한 비유적 표현으로, software engineering 관점에서 기존의 시스템에 축적되어서 새로운 기능을 추가하거나, 변경 / 유지보수 할 때 이를 어렵게 만드는 요소들로, 금융에서의 `부채`와 비슷한 개념으로 바라볼 수 있습니다. 

~~기술 부채는 대략 다음의 한 짤로 요약됩니다...~~

![](https://miro.medium.com/max/863/1*55Jn564sqMdKT39XIW3uxQ.png)

소프트웨어 엔지니어링에서와 마찬가지로 라이브 시스템에 ML 을 적용하기 시작하면서 ML system 에서도 `개발과 배포에 걸리는 시간은 상대적으로 빠르고 저렴하지만, 이를 유지보수하는 것은 어렵고 비용이 많이 들게되는` 현상이 광범위하게 발생하고 있습니다.

특히 ML system 에서는 `전통적인 코드를 유지 보수하는 문제에 추가로 ML에 특정된 이슈들로 인한 기술적 부채들도 발생`합니다. 그런데 이런 기술 부채들은 코드 레벨보다는 시스템 레벨에 존재하기 때문에 감지하기 어렵습니다. ML 시스템에서 코드 외 `시스템에서 흘러가는 데이터` 때문에 전통적인 추상화 경계(abstraction boundaries)가 깨지기 쉽고, 코드 레벨의 기술부채를 갚는 전형적인 방법들 (e.g. 리팩토링, 테스트코드 작성 등)로는 ML system의 부채를 해결하는데 충분하지 않습니다.

이 논문에서는 ML 기술 부채가 빠르게 축적될 수 있는 ML system 내의 시스템 레벨의 상호작용과 인터페이스에 집중하여 ML 기술 부채를 발생시키는 요소들에 대해 제시하고 있습니다.

## 2. Complex Models Erode Boundaries

전통적인 소프트웨어 엔지니어닝에서는 캡슐화와 모듈 디자인를 사용한 강한 추상화 경계(abstraction boundaries)를 통해 유지보수가 가능한 코드를 만들고, 독립된 작은 변화와 개선을 만들기 쉽게 합니다. 엄격한 추상화 경계는 인풋과 아웃품에 대한 일관성을 가지게 합니다.

그러나 ML 시스템에서는 `설계했던 동작이 외부의 데이터 의존성을 제외하고 소프트웨어 로직만으로는 효과적으로 표현 될 수 없기 때문에` 엄격한 추상화 경계를 적용하기 어렵고, 이로 인해 심각한 기술 부채를 만들 수 있습니다.

### Entanglement

ML 시스템에서는 `입력값들이 서로 복잡하게 얽혀있기 때문에, 독립적으로 개선 하기 어렵`습니다. 예를 들어 x_1, ... , x_n 의 feature를 사용하는 모델에서 만약 x_1 의 입력 분포를 바꾸는 경우, 나머지 모든 n-1개의 feature 들의 가중치, 사용 여부 등이 모두 바뀌게 됩니다. 새로운 n+1 번째 feature 를 추가하거나, 특정 feature x_j 를 제거하는 경우도 마찬가지 입니다.

이 논문에서는 이러한 경우를 CASE (`Changing Anything Changes Everything`) 원칙으로 명명했습니다. CASE 원칙은 입력값에 대해서 뿐아니라, 하이퍼파라미터, 학습에 필요한 설정들, 샘플링 방법등 ML system 의 모든 가능한 작업에 적용될 수 있습니다.

(해결책 TBD)

### Correction Cascades

문제 A 를 풀기 위한 모델 m_a 가 존재할 때, 문제 A와 약간만 다른 문제 A'에 대한 해결책이 필요할 때, 빠른 해결책으로 모델 m_a를 입력값으로 하고 일부분만 재학습해서 사용하는 m_a'를 만들 수 있습니다.

그러나, 이렇게 만들어진 모델의 경우 모델 m_a에 대한 의존성을 가지고 있어 차후에 모델을 개선하기 위해 분석하는 비용이 많이 들게됩니다.

(해결책 TBD)

### Undeclared Consumers

> SE에서도 동일하게 발생하는 문제 같은데

모델 m_a를 통한 예측 결과가 다른 시스템에서 널리 접근가능하게 만들어진 경우, 접근 제한이 없는 경우 선언하지 않은 소비자들이 모델의 출력값을 사용하게 되는 경우가 발생합니다.

## 3. Data Dependencies Cost More than Code Dependencies

전통적인 소프트웨어 엔지니어링에서 코드를 복잡하게 하고 기술 부채를 발생시키는 주요 요소로 `Dependency dept`를 꼽고 있습니다. ML 시스템에서도 이와 비슷하게 `data dependency` 가 기술 부채를 쌓고 있으며, 더 감지하기도 어렵다는 사실을 발견했습니다. 코드 의존성의 경우 compiler, linker 등 static analysis 를 통해 확인할 수 있습니다. 데이터 의존성의 경우도 의존성을 파악할 수 있는 도구가 없다면, 대규모의 해결하기 어려운 데이터 의존성을 쌓기 쉽습니다.

### Unstable Data Dependencies

모델에서 `다른 시스템에서 만든 출력값을 모델의 입력값으로 사용하게 되는 경우가 많은데, 이런 경우 시간의 흐름에 따라 질적으로, 양적으로 변화하는 unstable 한 입력값`이 될 수 있습니다. 예를 들어 다른 모델의 출력 값을 입력값으로 사용하는 경우 해당 모델이 변경됨에 따라 출력값이 암묵적으로 바뀔 수 있습니다. 이러한 입력값의 변화는 이를 사용하는 시스템에서 감지하기 어렵기 때문에 위험합니다.

이에 대한 한 가지 해결 전략으로는 `데이터에 대해 versioned copy 를 만들어 데이터 의존성을 해결하는 방법`이 있습니다.

> 논문에는 없지만, 지난번에 사용해보고 작성한 데이터 버전 관리 툴 [DVC](https://inahjeon.github.io/dvc/) 글 참고

### Underutilized Data Dependencies

활용도가 낮은 데이터들.

- Legacy Features
- Bundled Features
- ϵ-Features
- Correlated Featuress

### Static Analysis of Data Dependencies

전통적인 코드에서는 컴파일러와 빌드 시스템에서 의존성 그래프에 대해 정적 검사를 수행할 수 있습니다. 데이터 의존성에 대한 정적 분석 도구는 자동화된 feature 관리 같은 도구를 예를 들 수 있고, 아직 그리 많이 있지는 않습니다.

## 4. Feedback Loops

Live ML 시스템에서 한 가지 중요한 특성은 `시간이 흐름에 따라 시스템의 동작이 다시 모델에 영향을 끼치게 되는 것`입니다. 이러한 특성은 모델이 배포되기 전에는 시스템이 어떻게 동작할 지 예측하기 어려운 `analysys debt` 를 발생시킬 수 있습니다.

> 모델이 데이터에 영향을 끼치고, 다시 그 데이터가 모델에 영향을 끼치는 건가 봉가

- **Direct Feedback Loops**: 어떤 모델이 해당 모델에 사용할 미래의 학습 데이터를 선택하는데 직접적으로 영향을 끼질 수 있습니다. 이러한 문제는 supervised algorithm 에서 보통 bandit algorithm 을 통해 해결합니다.

> 뭔말인지 모르겠음.

- **Hidden Feedback Loops**: 직접적인 피드백 루프는 분석하기 어렵지만, 적어도 통계학적으로 해결책을 찾아볼 수는 있습니다. 더욱 더 어려운 케이스는 실세계에서 간접적으로 서로 영향을 주고 있는 시스템들에서 발생하는 숨어있는 피드백 루프입니다. 예를 들어 웹 페이지의 facet을 결정하는 두 시스템 (보여줄 제품을 선택하는 시스템과 제품에 관련된 리뷰를 선택하는 시스템) 에서 한 시스템에서의 개선 사항은 다른 시스템에 영향 (유저가 특정 제품을 더 많이/적게 클릭하는 등)을 끼칠 수 있습니다. 이러한 `숨은 피드백 루프가 완전히 독립된 시스템 사이에서 발생`한다는 사실을 유념해야 합니다.

## 5. ML-System Anti-Patterns

![](https://user-images.githubusercontent.com/16538186/74599412-be9d1880-50c4-11ea-919b-8d0971757405.png)

Real-world ML system 에서 실제 학습이나 예측에 사용되는 "ML 코드"가 차지하는 비중은 매우 적고, ML system 을 위한 요구되는 인프라는 매우 복잡합니다.

아래에서는 이러한 ML 시스템에서 반드시 피하거나, 리팩토링 되어야하는 몇 가지 `system-design anti-pattern` 들에 대해 다룹니다.

### Glue Code.

ML 연구자들은 독립된 package 형식으로 솔루션을 개발하려는 경향이 있고, 오픈소스 형태로 다양하게 존재하고 있습니다. 이러한 package들을 사용하게 될 경우, 보통 이 package를 사용하기 위해 데이터 입출력을 위한 대량의 지원 코드를 작성하게 됨으로써 `glue code system design pattern` 으로 빠지게 합니다. Glue code 는 특정 package를 사용하도록 시스템을 종속시키게 되어 장기적으로는 비용이 큽니다.

Glue code 문제를 해결하기 위한 한 가지 전략은 이러한 `black-box package 를 common API로 wrapping 하는 방법` 이 있습니다. Common api 로 wrapping 하면 package의 변화에 따른 비용을 줄이고, 재사용성을 더 높일 수 있습니다.

### Pipeline Jungles.

Glue code의 특수한 케이스로, 데이터 전처리 단계에서 `pipeline jungles` 가 발생할 수 있습니다.

### Dead Experimental Codepaths

### Abstraction Debt

### Common Smells.

- Plain-Old-Data Type Smell.
- Multiple-Language Smell.
- Prototype Smell.

## 6. Configuration Debt

(TBD)

## 7. Dealing with Changes in the External World

ML system 이 매력적인 한 가지 이유는 외부 세계와 직접적으로 상호작용한다는 것입니다. 경험적으로, 외부 세계는 안정적인 경우가 매우 드물기 때문에, `외부 환경에 대한 변화로 인한 유지보수 비용이 지속적으로 발생`합니다.

### Fixed Thresholds in Dynamic Systems.

ML system에서 스팸 메일인지, 정상 메일인지 예측하여 표시하거나, 특정 광고를 유저에게 표시할 지 말지 등 특정 ML 모델이 어떤 동작을 수행하게 하기 위해 `decision threshold` 값 를 선택하는 경우가 많습니다. 한 가지 전통적인 방법은 가능한 임계값 범위 내에서 precision / recall 과 같은 tradeoffs 를 고려하여 하나 택하는 방법이 있습니다. 

그런데 이러한 `임계값들을 대부분 수동적으로 설정하는 경우`가 많습니다. 만약 모델이 새로운 데이터를 입력으로 받게되는 경우, 기존의 수동적으로 설정한 임계값은 유효하지 않을 수 있습니다. 또한, 많은 모델들의 임계값들을 수동으로 변경하는 일은 시간이 오래걸리고 다루기 힘든 작업입니다.

이러한 문제를 해결하기 위한 한 가지 전략은, `검증 데이터 셋을 통해 임계값을 학습하게 하는 방식` 을 사용할 수 있습니다.

### Monitoring and Testing.

동작하고 있는 시스템에서 개별 component 들에 대해 unit test 및 end-to-end 테스트를 수행하는 것은 중요한 작업이지만, 변화하는 실 세계 환경에서 이러한 테스트들은 해당 시스템이 의도한 대로 잘 동작하고 있다는 것을 증명하기에는 불충분할 수 있습니다.

장기적인 시스템 안정성에 대한 관점에서 자동화된 실시간 모니터링은 매우 중요합니다. 중요한 것은 `무엇을 모니터링 할 것인가?` 입니다. ML system 에서는 테스트 해야하는 값이 명확하게 주어지는 경우가 많습니다. 모니터링 해볼 좋은 시작점으로 아래의 요소들에 대해 제시합니다.

- **[Prediction Bias](https://developers.google.com/machine-learning/crash-course/classification/prediction-bias)**: ML system 이 의도한 대로 잘 동작하고 있다면, 시스템에서 예측하는 label 의 분포가 관측한 label 의 분포가 동일해야합니다. `Prediction bias 를 측정하는 방안은 갑작스럽게 변경되는 외부의 변화를 감지하는데 유용`하게 쓸 수 있습니다. 다양한 차원에서 prediction bias를 나눔으로써, 이슈를 빠르게 분리하고, 자동화된 알림으로 사용할 수 있습니다.

- **Action Limits**: ML system 에서 스팸 문자를 마킹하는 등의 특정 동작을 수행하는 경우, `동작에 대한 limit을 설정하는 방법`은  sanity check 로 유용합니다. 만약 시스템에서 특정 동작에 대한 limit 을 넘어서는 경우, 자동 알림을 보내서 수동적으로 검사해볼 수 있습니다.

- **Up-Stream Producers**: ML system에서 입력 데이터를 다양한 up-stream producer 들로 부터 수급받게 되는 경우가 많습니다. 이러한 up-stream process 들은 ML system의 요구사항을 포함한 서비스 레벨의 목적을 달성할 수 있도록 철저하게 모니터링되고 테스트되어야 합니다. 이러한`up-stream 들의 failure에 대한 alert 또한 반드시 ML system의 control 전략에 포함되어야하고, 비슷하게 ML system 에서의 failure 역시 down-stream consumer 들에게 전파되어 down-stream consumer 들의 control 전략에 포함`되어야 합니다.

외부의 변화는 실시간으로 일어나기 때문에, 이에 대한 반응 또한 실시간으로 발생합니다. 사람이 직접 개입해서 대응하는 것이 보통이지만, 빠르게 해결되어야하는 문제에 대해서는 취약할 수 있습니다. 사람이 직접 개입하지 않고 자동으로 변화에 대해 대응할 수 있는 시스템을 만드는 것은 투자할만한 가치가 있습니다.

> 그런 시스템을 만드는 게 가능한가? 단순한 조치는 취할 수 있을듯

## 8. Other Areas of ML-related Debt

그 밖에 ML 관련 기술 부채들에 대한 내용입니다.

### Data Testing Debt.

데이터가 ML system 의 코드를 변경시키는 경우, 해당 코드에 대해서 반드시 테스트하고, 입력 데이터 또한 테스트하는 것은 매우 당연합니다. `기본적인 sanity 체크 및 입력 값의 분포의 변화에 대해 모니터링하기 위한 잘 설계된 테스트` 가 유용하게 쓰일 수 있습니다.

### Reproducibility Debt.

Scientist 로써, 실험을 다시 수행하고 `비슷한` 결과물을 얻는 것은 중요합니다. 그러나 real-world ML system 에서 Randomized 알고리즘의 사용, 초기 조건에 따른 의존성 등의 문제로 `strict reproducibility` 를 가지도록 설계하는 것은 더욱 더 어렵습니다.

### Process Management Debt.

이 논문에 소개된 대부분의 사례들은 한 가지 모델을 유지보수 하기 위한 비용에 대해 말하고 있습니다. 그러나 성숙한 시스템의 경우는 수 십, 수 백개의 모델들이 동시에 실행됩니다. 이런 성숙한 시스템의 경우, `많은 비슷한 모델들의 congifuration 값들을 안전하게 자동적으로 변경하는 문제`, `비즈니스 우선순위가 다른 모델등 사이에서 리소스를 할당하고 조율하는 문제`, `production 데이터 파이프라인의 흐름을 시각화하고 blocker 들을 감지하는 문제` 등 넓은 범위에 걸쳐 해결해야 할 다양하고 중요한 문제들이 있습니다. 또한, `장애 시 복구를 위한 tool을 개발`하는 것도 매우 중요합니다.

### Cultural Debt.

ML research 와 engineering 사이에 큰 장벽이 있는 경우, 이러한 장벽은 장기적인 관점에서 생산성을 저하시킬 수 있습니다. `팀 차원에서 쓸모없는 feature 의 개수를 줄이고, 시스템의 복잡도를 낮추고, 재현성과 안정성을 개선하고, 시스템을 모니터링하는 것에 대해 보상하는 문화를 만드는 것이 중요` 합니다.

## 9. Conclusions: Measuring Debt and Paying it Off

기술 부채는 유용한 표현이지만, 기술적 부채에 대해 측정할 수 있는 구체적인 metric 까지는 제시하고 있지 않습니다. 다음과 같은 질문들을 통해 기술적 부채가 얼마나 존재하는지 대략적으로 파악해 볼 수 있습니다.

> - 새로운 알고리즘적 접근이 얼마나 쉽게 full scale 에서 테스트 가능한가?
> - What is the transitive closure of all data dependencies? (~~무슨 말인지 모르겠음~~)
> - 시스템에 적용되는 새로운 변화에 대한 임팩트를 얼마나 정확하게 측정할 수 있는지?
> - 특정 모델이나 입력값에 대한 개선이 다른 것들의 성능을 저하시키는지?
> - 새로운 팀 멤버가 얼마나 빨리 업무에 적응할 수 있는가?

이 논문을 통해 유지보수 가능한 ML 시스템 개발을 위한 더 나은 추상화, 테스팅 기법, 디자인 패턴들이 나오길 바라고, 논문에서 제시하는 가장 중요한 시사점은 `engineer 와 researcher 모두 기술 부채에 대해 경계하고 고민해야한다는 점` 입니다. 

정확도(accuracy)에서 작은 개선점을 얻을 수 있는 특정 research 솔루션이 전체 시스템의 복잡도을 매우 높이는 경우는 좋은 practice 가 될 수 없습니다.

ML 관련된 기술 부채를 줄이기 위해서는 팀 문화를 바꾸는 차원의 특별한 노력이 필요합니다. 장기적으로 성공적인 ML팀이 되기 위해 이러한 노력에 대해 인식하고 우선순위하하고, 보상하는 것이 중요합니다.

