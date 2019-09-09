---
layout: post
title:  "Google Cloud Summit 2018"
date:   2018-11-04 23:14:00
tags: [conference]
comments: true
---

# 2018-10-25-google-cloud-summit-review

# 느낀점

생각보다 규모가 매우 큰 [행사](https://cloudplatformonline.com/2018-Summit-Korea-Agenda.html)였다. 여러 클라우드 서비스 컨설팅 업체나 제휴사들이 많이 참석하고, 제품을 홍보하는 부스가 많았다. 구글에서도 클라우드 서비스를 홍보하거나, 교육 프로그램이 많았다.

그 중에서 quicklab 이라는 교육 프로그램을 통해서 cloudML을 사용해서 머신러닝 모델을 학습시키고 배포하는 프로그램을 체험해봤다. 주어진 github 코드와 데이터 셋이 있어서 메뉴얼에 나온 명령어만 그대로 따라하면 되어 매우 쉬웠다. 쉬웠던 만큼 크게 머리에 남은건 없었던 것 같지만, 교육 프로그램에서 cloudML 서비스를 한번 체험시키는 목적은 달성한 것 같다.

전체적으로 구글 클라우드 짱짱맨이니까 많이 쓰세요! 였지만, Data Scientist로써 GCP를 잘 활용하고 있고 좋아하기 때문에 어떤 것들을 도입해 볼수 있을지 product 구축 사례를 잘 들었다.

# 세션: 데이터 사이언티스트를 위한 Cloud AI Platform 소개

한기환 - Cloud Consultant, Google Cloud 

이용운 - 팀장, nARC (netmarble AI Revolution Center) 콜럼버스실 데이터인텔리전스팀, 넷마블

## 1. **기업의 목소리**

**AI 채택까지의 먼 길**

- 40%이상의 기업이 AI를 통한 이익에 대해 확신이 없음 (ROI가 나올 것인가)
- 20%의 기업은 AI도입을 위해 노력중

**누가 AI를 사용할 수 있나?** 

- 2100만명: 개발자
- 100만명 이하: 데이터 사이언티스트
- 1000명: 딥러닝 연구원

**컴퓨팅 성능의 중요성** 

- 최근 AI발전의 80%는 컴퓨팅 성능 향상에 따른 것임.
- 85%의 기업이 다중 클라우드 전략을 사용하고 있거나 채택 과정에 있음.

## **2. 구글의 비전: 구객과 파트너가 AI를 쉽고 빠르고 유용하게 사용할 수 있도록 대중화하자**

CloudAI 소개 

- 초보자: Cloud ML Engine
- 능숙자: BigQuery ML
- 전문가: AutoML 및 CloudML API

숙련도에 따라 누구나 활용 가능한 서비스들이 있음.

**왜 구글을 사용해야하는지?** 

- 규모
- 속도
- 품질
- 접근가능성

데이터 서비스쪽은 google cloud 를 가장 선호했음 (통계)

## **3. 서비스 소개**

- Cloud Dataprep
    - 분석 또는 ML 프로젝트를 위한 데이터를 시각적으로 준비
- Data Studio를 사용한 협업 BI
    - 원클릭 시각화
- BigQuery ML
    - 데이터를 이동하지 않고, 계획된 ML 실행
    - 모델 선택 및 하이퍼 튜닝 자동화
    - BigQuery의 SQL로 모델 튜닝을 반복하여 개발 속도 향상
- Cloud Datalab
    - 빅쿼리 데이터와 연결된 notebook
- 클라우드 ML 엔진
    - HyperTune 으로 모델 최적화 가속: 자동 하이퍼 파라미터 튜닝 서비스  ******
- ***Kubeflow***
    - Kubernetes용 클라우드 네이티브 ML
    - 멀티 클라우드 전략을 사용하는 기업에서 쉽게 이동가능하도록 해줌.
    - 텐서플로, 파이토치 등 즉시 사용할 수 있도록 지원
    - Kubernetes가 종속성, 리소스 관리
    - 원하는 곳에서 실행. 스왑기능 및 확장 가능.

## **4. 고객 사용 사례: 넷마블**

BigQuery - 데이터레이크 

DataPrep - 데이터 전처리, 대강 데이터 간보기 

Jupyter - 피쳐 엔지니어링, 피쳐 셀렉션 진행, 

Data flow - 분산처리해서 학습 데이터 생성 

Cloud Storage - 학습 데이터 저장 

ML 엔진 - 학습 정확도 모니터링, 하이퍼 파라미터 튜닝
