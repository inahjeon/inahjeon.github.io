---
layout: post
title:  "Meet Michelangelo: Uber’s Machine Learning Platform (번역) (2)"
date:   2019-04-08 01:03:00
tags: [machine learning, uber, machine learning platform, Michelangelo]
comments: true
---

### 참고자료 

[Meet Michelangelo: Uber’s Machine Learning Platform](https://eng.uber.com/michelangelo) - Uber Machine Learning Platform team의 Jeremy Hermann (Engineering Manager), Mike Del Balso (Product Manager)

## Evaluate models

모델들은 주로 문제를 해결에 가장 좋은 모델을 생성하는 feature 집합, 알고리즘, hyper-parameter들을 식별하기 위한 체계적인 탐색 프로세스의 일부로 학습됩니다. 주어진 usecase에 이상적인 모델에 도달하기 전에 목표를 달성하지 못하는 수 백개의 모델을 학습하는 일은 드문 일이 아닙니다. 비록 최종적으로 제품에 사용되지 않지만, 이러한 모델들의 성능은 엔지니어들이 가장 좋은 모델에 대해 configuration을 할 수 있도록 가이드 해줍니다. 이러한 학습된 모델들을 지속적으로 트래킹(예: 누가 그 모델들은 언제 어떤 데이터셋과 hyper-parameter로 학습했는지 등) 하고, 평가하고, 다른 모델들과 비교하는 것은 수많은 모델들을 다루는 환경에서 일반적으로 큰 어려움이 있고, 현재 플랫폼에는 큰 가치를 더할 수 있는 기회입니다.

Michelangelo에서 학습된 모든 모델들에 대한 버전 객체를 우리의 Cassandra의 모델 저장소에 저장합니다. 버전 객체는 다음의 내용을 포함하고 있습니다.

- 누가 모델을 훈련 시켰는가?
- 학습 작업의 시작 및 종료 시간
- 전체 모델 구성 (사용된 feature들, hyper-parameter 값 등)
- 학습 및 테스트 데이터셋에 대한 참조
- 각 feature들의 분포와 중요도
- 모델 정확도 지표
- 각 모델 타입에 대한 표준 차트와 그래프 (예: 이진 분류기에 대한 ROC 곡선, PR 곡선, confusion matrix)
- 모델에서 학습된 전체 paramter들
- 모델 시각화를 위한 요약 통계

이러한 정보는 웹 UI를 통해 사용자들이 쉽게 사용할 수 있고, API를 통해 개별 모델들의 세부 사항에 대한 검사와 하나 이상의 서로 다른 모델들을 비교할 수 있습니다.

## Model accuracy report

회귀 모델에 대한 모델 정확성 보고서는 표준 정확도 지표와 차트들을 보여줍니다. 분류 모델들은 Figure 4나 5와 같이 다른 집합을 표시합니다:

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image9.png)
Figure 4: Regression 모델 리포트는 regression과 연관된 성능 지표를 보여줍니다.

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image10.png)
Figure 5: 이진 분류의 성능 리포트는 분류와 연관된 성능 지표를 보여줍니다.

### Decision tree visualization

중요한 모델 타입의 경우, 우리는 모델러들이 왜 모델이 정상적으로 작동하는지 이해하고 필요한 경우 디버깅할 수 있도록 돕기 위해 정교한 시각화 도구들을 제공합니다. Decision tree 모델들의 경우, 우리는 사용자들이 각각의 개별 tree들을 탐색하여 전체적인 모델에 대한 상대적인 중요도, 분할 지점, 특정 tree에 대한 개별 feature들의 중요도, 각 분할 지점에서의 데이터 분포를 확인할 수 있게 합니다. 아래 Figure 6에서 보듯이, 사용자가 feature 값을 명시할 수 있고, 그에 따라 decision tree에서 활성화되는 경로, tree 별 예측, 전체적인 모델의 예측에 대해 시각화합니다:

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image12-2-1024x577.png)
Figure 6: Tree 모델들은 강력한 tree 시각화로 탐색할 수 있습니다.

### Feature report

Michelangelo는 개별 feature들에 대해 모델에서의 중요도 순으로 부분적 의존 관계 그래프와 분포 histogram을 보여주는 feature 보고서를 제공합니다. 아래와 같이, 두 가지 feature를 선택하면 사용자는 양방향 부분적 의존 관계 다이어그램을 통해 feature들의 상호작용에 대해 이해할 수 있습니다:

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image11.png)
Figure 7: Feature들, 모델에서의 영향력, feature 간 상호작용을 탐색해 볼 수 있습니다.

## Deploy models

Michelangelo는 UI 또는 API를 통해 모델 배포 관리를 위한 end-to-end 지원을 제공하고, 모델을 배포할 수 있는 세 가지 모드를 지원합니다:

**Offline deployment.** 
이 모델은 offline 컨테이너에 배포되고, on demand 또는 반복되는 스케줄링에 따른 배치 예측 작업을 위해 Spark job에서 실행됩니다.

**Online deployment.** 
이 모델은 online 예측 서비스 클러스터(일반적으로 load balancer 뒤에 수 백개의 머신을 포함하고 있는)에 배포됩니다. Client는 RPC 호출과 같은 개별 또는 배치 예측 요청을 보낼 수 있습니다.

**Library deployment.** 
우리는 다른 서비스에서 라이브러리 형태로 내재화되어 Java API로 호출될 수 있는 모델을 serving 컨테이너에 배포하려고 합니다. (아래 Figure 8에는 표시되어 있지 않지만 online deployment와 유사하게 동작합니다)

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image6.png)
Figure 8: 모델 서빙을 위해 모델 저장소로부터 모델이 online과 offline 컨테이너에 배포됩니다.

모든 케이스에서, 배포에 필요한 모델 결과물(metadata 파일, 모델 parameter 파일, 컴파일된 DSL 표현식들)은 ZIP 파일로 압축되고, 우리의 표준 코드 배포 인프라를 사용하여 Uber의 데이터 센터를 통해 연관있는 host에게로 복사됩니다. 예측 컨테이너들은 자동적으로 디스크로부터 새로운 모델을 불러오고 예측 요청을 처리하기 시작합니다.

많은 팀들은 Michelangelo의 API를 활용하여 주기적인 모델 재학습과 배포를 위한 자동 스크립트를 보유하고 있습니다. UberEATS의 배달 시간 모델의 사례에서는, 데이터사이언티스트들과 엔지니어들이 웹 UI를 통해 학습과 배포를 수동으로 트리거 합니다.

## Make predictions

한번 모델이 배포되고 서빙 컨테이너에 로드되면, 모델들은 데이터 파이프라인이나 client 서비스로 부터 직접 불러온 feature 데이터를 기반으로 예측 작업을하는 데 사용됩니다. 원본 feature들은 원본 feature를 수정하거나, Feature Store에서 부가적인 feature를 추가할 수 있는 컴파일된 DSL 표현식에 전달됩니다. 그리고 최종 feature vector가 계산되어 스코어링을 위해 모델에 전달됩니다. Online 모델의 경우, 예측 결과는 네트워크를 통해 Client 서비스에 반환됩니다. Offline 모델의 경우, 예측 결과는 Hive에 다시 쓰여지고, 배치 작업에 사용되거나 사용자가 SQL 기반 쿼리 툴을 사용해서 접근할 수 있습니다.

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image3.png)
Figure 9: Online과 Offline 예측 서비스에서 feature vector 셋을 예측하는데 사용합니다.

## Referencing models

하나 이상의 모델을 주어진 서빙 컨테이너에 동시에 배포할 수 있습니다. 이것은 이전 모델을 새로운 모델로 안전하게 변경하는 작업과 모델 간 A/B테스팅을 가능하게 합니다. 서빙 시점에서, UUID와 배포중에 명시된 부가적인 태그(또는 별칭)를 통해 모델이 식별됩니다. Online 모델의 경우, 클라이언트 서비스가 사용하고자 하는 모델의 UUID나 모델 태그와 함께 feature vector를 전송합니다. 태그를 사용할 경우, 컨테이너는 해당 태그에 가장 최근에 배포된 모델을 사용하여 예측 결과를 생성합니다. 배치 모델의 경우 모든 배포된 모델들은 개별 배치 데이터셋에 대해 점수를 매기는데 사용되고, 예측 결과들이 모델 UUID와 부가적인 태그를 포함하고 있어 사용자가 적절하게 결과를 필터링할 수 있습니다.

만약 이전 모델을 대체하기 위해 새로운 모델을 배포할 때 두 모델이 같은 signature를 가졌다면(예: 동일한 feature 셋을 가졌을 경우), 사용자들은 새로운 모델을 이전 모델과 같은 태그로 배포할 수 있고, 컨테이너들은 새로운 모델을 즉시 사용하기 시작합니다. 이것은 사용자들이 그들의 client 코드를 수정하지 않고도 모델을 갱신할 수 있게 해줍니다. 사용자들은 또한 이전 모델에서 새로운 모델로 점진적으로 트래픽을 전환하기 위해 client나 중간 서비스에서 UUID만으로 새로운 모델을 배포하고 configuration을 변경할 수 있습니다.

모델 간의 A/B 테스팅을 위해, 사용자들은 간단하게 UUID나 태그를 통해 비교할 모델들을 배포할 수 있고, 클라이언트 서비스에서 Uber의 실험 프레임워크를 사용하여 트래픽의 일부를 각 모델에 보내고 성능 지표를 트래킹할 수 있습니다.

## Scale and latency

머신러닝 모델들이 stateless이고 무엇도 공유하지 않기 때문에, online이나 offline 서빙 모드 모두에서 스케일 아웃하기 쉽습니다. Online 모델의 경우, 우리는 간단하게 더 많은 호스트를 예측 서비스 클러스터에 추가하고, load balancer로 네트워크 부하를 퍼뜨릴 수 있습니다. Offline 예측의 경우, 우리는 더 많은 Spark 실행작업을 추가하고 Spark로 병렬처리를 관리할 수 있습니다.

Online 서빙 응답시간은 모델이 Cassandra feature store에 저장된 feature를 가져오는 것과는 관계없이 모델 타입과 복잡도에 달려있습니다. Cassandra에 저장된 feature를 필요로 하는 모델의 경우, 주로 5 밀리초 이내의 P95의 응답시간 정도 소요됩니다. Cassandra의 feature를 필요로 하는 모델의 경우, 주로 10 밀리초 이내의 P95의 응답시간을 가집니다. 가장 높은 트래픽의 모델은 현재 초당 250,000개의 예측을 수행하는 모델들입니다.

## Monitor predictions

모델이 학습되고 평가될 때, 기록 데이터가 항상 사용됩니다. 특정 모델이 미래에도 잘 동작할 지 확인하기 위해, 데이터 파이프라인이 계속 정확한 데이터를 전송하고 모델이 더 이상 정확하지 않게 production 환경이 변하지 않았는지 모델의 예측 결과를 모니터링 하는 것은 중요합니다.

이 문제를 해결하기 위해 Michelangelo는 자동으로 로그를 기록하고, 선택적으로 예측 결과 비율을 저장하여 데이터 파이프라인에서 생성된 실제 관측 결과(또는 label)와 결합합니다. 이러한 정보를 통해 우리는 실시간으로 모델의 정확도에 대한 지표를 생성할 수 있습니다. 회귀 모델의 경우, 아래 그림과 같이 우리는 예측 결과에 대한 R-squared/coefficient, root mean square logarithmic error (RMSLE), root mean square error (RMSE), mean absolute error 지표를 Uber의 시계열 모니터링 시스템에 발송하여 사용자들이 시간에 따라 차트를 분석하고 특정 threshold 값으로 알람을 설정할 수 있도록 합니다.

![](https://1fykyq3mdn5r21tpna3wkdyi-wpengine.netdna-ssl.com/wp-content/uploads/2017/09/image8.png)
Figure 10: 예측 결과가 샘플링되고 관측 결과와 비교하여 모델의 정확도 지표를 생성합니다.

## Building on the Michelangelo platform

앞으로 몇 달 동안, 우리는 고객 팀과 Uber 비즈니스 전반의 성장을 지원하기 위해 기존 시스템을 아래와 같은 기능들에 대해 계속 확장하고 강화할 계획입니다. 플랫폼 계층이 성숙 해짐에 따라 우리는 머신러닝의 민주화를 추진하고 비즈니스 요구를 보다 잘 지원하기 위해 더 높은 수준의 도구와 서비스에 투자 할 계획입니다.

**AutoML.**
이것은 주어진 모델링 문제에서 모델이 최고의 성능을 내도록 하는 configuration(알고리즘, feature 셋, hyper-parameter 값들 등)들을 자동적으로 검색하고 발견하는 시스템이 될 것 입니다. 이 시스템은 또한 모델에 필요한 feature들과 label들을 생성하기 위해 자동적으로 production 데이터 파이프라인들을 생성할 것 입니다. 우리는 우리의 Feature Store, 균일화된 오프라인과 온라인 데이터 파이프라인들, hyper-parameter 검색으로 이 문제에 대한 큰 부분들을 이미 해결했습니다. 우리는 초반의 데이터 사이언스 업무를 AutoML을 통해 가속화하려고 합니다. 이 시스템은 데이터 과학자들이 label들과 목적함수를 정의하고, Uber의 데이터를 보안적으로 안전하게 사용하여 문제에 대한 최적을 답을 찾을 수 있도록 합니다. 목표는 똑똑한 도구로 데이터 과학자들을 일을 손쉽게 만들어서 생산성을 높이는 것입니다.

**Model visualization.**
모델을 이해하고 디버깅하는 것은 특히 딥러닝에서 점점 중요해지고 있습니다. 우리가 트리 기반의 모델들을 시각화하는 도구로 몇가지 중요한 첫 스텝들을 밟고 있는 반면, 데이터 과학자들이 예측 결과를 믿고, 모델을 이해하고, 디버그하고 튜닝하기 위해 완료되어야 할 더 많은 니즈들이 남았습니다.

**Online learning.**
대부분의 Uber의 머신러닝 모델들은 직접적으로 Uber의 production 환경에 실시간으로 영향을 끼칩니다. 이것은 그들이 물리적 세계의 물체가 움직이는, 복잡하고 계속 변화하는 환경에서 동작을 실행한다는 의미입니다. 우리의 모델이 환경이 변화하더라도 정확성을 유지하기 위해, 우리는 변화에 맞게 모델을 변경해야합니다. 현재, 팀들은 규칙적으로 Michelangelo에서 그들의 모델을 재학습시킵니다. 이 use case에 대한 완벽한 플랫폼 솔루션은 손쉬운 모델 타입 갱신, 빠른 학습과 평가 구조 및 파이프라인, 자동화된 모델 검증 및 배포 그리고 섬세한 모니터링과 알림 시스템을 포함합니다. 큰 프로젝트일지라도, 초기 결과물은 online learning을 당장 시작하는 것이 뒤따라오는 잠재적인 이익을 얻을 수 있음을 나타냅니다.

**Distributed deep learning.**
증가하고 있는 Uber의 머신러닝 시스템들은 딥러닝 기술로 구현되어있습니다. 딥러닝 모델을 정의하고 반복시키는 사용자의 워크플로는 표준화된 워크플로와 충분히 달라 플랫폼의 고유한 지원이 필요합니다. 딥러닝 use case들은 일반적으로 많은 양의 데이터를 다루고, GPU같은 다른 하드웨어 사양이 필요하기때문에, 분산 학습에 대한 더 많은 투자와 유연한 자원관리 스택과 긴밀한 통합을 유도하고 있습니다.
