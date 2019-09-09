---
layout: post
title:  "CloudML Engine으로 모델 학습부터 배포까지"
date:   2019-01-14 01:37:00
tags: [gcp, cloud-ml-engine]
comments: true
---

## 0. CloudML engine 이란

![](https://www.dropbox.com/s/ctlgcb0ac0im467/cloudml.png?raw=1)

학습 데이터 셋과 모델만 작성하면 나머지 모델을 학습시키고, 학습된 모델 버전을 관리하고, production에 활용할 수 있도록 배포하는 등의 머신러닝 workflow에서 필요한 기능들을 제공해주는 서비스이다.

Cloud 머신러닝 모델 학습 환경을 찾고 있던 터라 한번 사용해 보았다.

## 1. 학습 dataset 준비

GCP 문서에서는 미리 제공된 census 데이터를 활용하는 예제가 있지만, 예제만 그대로 따라했을 때 잘 모르고 넘어가게 되는 부분이 있어서 직접 다른 데이터셋과 코드를 짜서 진행해보기로 했다.

학습 데이터는 kaggle titanic dataset으로 간단히 테스트해보기로 했다. (아무 이유 없고 그냥 바로 생각나서)

kaggle에 올려져있는 dataset은 kaggle api 를 활용해서 다운 받을 수 있다. 

1) kaggle api 설치

```shell
pip install kaggle
```

2) kaggle api 활용하기 위한 token key 등록

kaggle profile 기능 에서 kaggle.json 파일(username, kaggle_key 정보가 있는 파일)을 다운 받아서 `~/.kaggle/kaggle.json` 에 위치하도록 옮긴 후, 다음과 같이 파일 권한을 조정한다.

```shell
chmod 600 ~/.kaggle/kaggle.json
```

3) 데이터 셋 다운로드
```shell
kaggle competitions download -c titanic
```

titanic dataset이 이 다음과 같이 다운로드 된 것을 확인할 수 있다.

![](https://www.dropbox.com/s/0zvtyedgwytyzj6/dataset.png?raw=1)


## 2. 로컬에서 모델 만들고 학습 테스트

ml engine에 사용할 코드는 아래 git repository 있는 예제 코드들을 참고해서 작성했다.

[https://github.com/GoogleCloudPlatform/cloudml-samples](https://github.com/GoogleCloudPlatform/cloudml-samples)

아래 주소에서는 ml engine의 기본 코드 구조에 대한 설명을 확인할 수 있다.
[https://github.com/GoogleCloudPlatform/cloudml-samples/tree/master/cloudml-template](https://github.com/GoogleCloudPlatform/cloudml-samples/tree/master/cloudml-template)

- metadata.py: input data, target feature, unused features 등 주로 입력 데이터 셋에 대한 정의를 하는 파일이다.

- input.py: input 파일을 custom하게 읽고, 간단한 데이터 처리를 하는 곳이다.
- featurizer.py: feature engineering을 위한 곳
- model.py: 학습 모델을 정의하는 곳
- task.py: cloud ml engine의 명령어를 처리하는 인터페이스

예제 코드에서 나머지는 크게 수정하지 않았고, 데이터 정의와 전처리를 위해 `metadata.py`, `input.py` 만 titanic 데이터에 맞게 수정해서 사용했다.

cloud ml engine을 사용하는게 목적이라 feature는 5개 정도만 간단히 넣어서 학습을 진행했다.

로컬에서 작성한 모델은 다음과 같이 테스트해 볼 수 있다.

```shell
gcloud ml-engine local train \
    --module-name trainer.task \
    --package-path trainer/ \
    --job-dir output \
    -- \
    --train-files data/train.csv \
    --eval-files data/eval.csv \
    --train-steps 1000 \
    --eval-steps 100
```

학습에 사용한 `train.csv` 과 `eval.csv` 는 원본 `train.csv` 의 데이터를 나눈 데이터를 사용했다.

## 3. Cloud ML Engine으로 모델 학습하기

GCP에서 ML engine을 돌리기 위해 로컬에서 테스트했던 코드와 dataset을 Google cloud storage에 업로드 한다.

![](https://www.dropbox.com/s/9k54vamumihlzwt/cloud_storage.png?raw=1)

- data: 학습할 데이터
- models: 학습된 모델을 저장할 장소
- trainer: 학습 모델 코드

Cloud ml engine은 다음과 같이 job을 만들어서 작업을 큐에 넣어 실행시킬 수 있다.

```shell
gcloud ml-engine jobs submit training $JOB_NAME \
    --module-name trainer.task \
    --package-path trainer/ \
    --job-dir $OUTPUT_PATH \
        --runtime-version 1.8 \
    --region $REGION \
    -- \
    --train-files $TRAIN_DATA \
    --eval-files $EVALUATION_DATA \
    --train-steps 1000 \
    --eval-steps 100
```

작업을 실행시키면, ML엔진에서 job이 실행되고 실시간으로 log를 조회할 수 있다.

![](https://www.dropbox.com/s/dwoi6wssyz495g0/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202019-01-13%20%EC%98%A4%ED%9B%84%206.08.07.png?raw=1)
(작업이 최종적으로 완료된 모습)


`titanic_single_8` 이라는 jobname을 보면 유추할 수 있듯이 예제를 따라해보며 single machine으로 돌린 titanic 모델 학습 job을 7번째 실패하고 8번째 시도만에 드디어 성공했다는 의미이다...! :sob: 흙..

#### 성공한 모델

![](https://www.dropbox.com/s/1osu5p89sh5zcmx/trained_model.png?raw=1)

드디어 첫번째 버전의 titanic 예측 모델이 배포되었다.

## 4. 생성한 모델로 prediction 해보기

Cloud ML engine에 올려진 모델을 이용해서 특정 input data 의 결과값을 prediction 해볼 수 있다.

파일 형식은 `csv`, `json` 형태를 지원하고 있고, 다음과 같이 `test.json` 을 만들어서 prediction 해보았다.

```json
{"Pclass": 3,"Sex": "male","Age": 0,"SibSp": 0,"Embarked": "Q"}
{"Pclass": 1,"Sex": "female","Age": 20,"SibSp": 0,"Embarked": "Q"}
```

여기서 처음에 한참 헤멨었는데, 입력 형식이 요상하다. -_-

json format이라고 되어있는데, **line마다 json input 데이터 1개가 들어가는 형식** 이다. input 파일 형태에 대해 문서에 명확하게 잘 나와있지 않아서 github 예제 코드에서 참고해서 만들었다. :cry:

그리고 모델에서 학습에 사용하지 않는 feature column key를 넣었을 때, 인식이 안되어서 에러가 났다. :thinking_face: (단순 예제 파일을 고쳐서 돌려서 여러가지 input 처리들이 잘 안되어 있는 탓일까..)

어찌됬든 `test.json` 과 만든 모델로 다음과 같이 prediction을 실행해 볼 수 있다.

#### prediction
```shell
gcloud ml-engine predict --model titanic101  --version titanic1 --json-instances test.json
```

#### 예측결과

![](https://www.dropbox.com/s/hzkihb8kqo9zpii/prediction.png?raw=1)

위와 같이 예측된 class 와 확률값에 대해서 확인해 볼 수 있다.

## 5. 간단한 서버로 ML서비스 만들어보기

이제 최종적으로 Google cloud ML API를 활용해서 간단한 머신러닝 서비스를 만들어보자.

서버 라이브러리는 python flask를 활용했다.

```
google-api-python-client==1.6.2
google-auth
google-auth-httplib2
flask
ujson
```

필요한 라이브러리를 설치하고, 다음과 같이 간단한 서버를 작성했다.

app.py

```python
import ujson
from google.oauth2 import service_account
from flask import Flask, jsonify, request
from googleapiclient import discovery
from .meta import GOOGLE_APPLICATION_CREDENTIALS, PROJECT_ID, MODEL_NAME, \
    MODEL_VERSION


def get_google_api_client():
    raw_credential = ujson.loads(GOOGLE_APPLICATION_CREDENTIALS)
    credentials = \
        service_account.Credentials.from_service_account_info(raw_credential)

    return discovery.build(
        'ml', 'v1', credentials=credentials
    )


GOOGLE_API_CLIENT = get_google_api_client()

app = Flask(__name__)


@app.route('/')
def index():
    return 'Hello Cloud ML Engine!'


@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    name = 'projects/{}/models/{}/versions/{}'.format(
        PROJECT_ID, MODEL_NAME, MODEL_VERSION
    )

    response = GOOGLE_API_CLIENT.projects().predict(
        name=name,
        body={'instances': data['data']}
    ).execute()

    if 'error' in response:
        raise RuntimeError(response['error'])

    return jsonify(response['predictions'])
```

(`GOOGLE_APPLICATION_CREDENTIALS` 값은 API 사용이 허가된 서비스 계정에서 키를 생성해서 받는 json을 이용하여 만들 수 있다.)

`PROJECT_ID`, `MODEL_NAME`, `MODEL_VERSION` 는 각각 사용했던 GCP 프로젝트 명, 학습된 모델명, 버전명을 넣어주면 된다.

그리고 다음과 같이 실행하면 기본적으로 localhost:5000 에 서버가 띄워진다.
```
FLASK_APP=app.py flask run
```

![](https://www.dropbox.com/s/7h2pt25k1wguoyd/server_run.png?raw=1)

사실 여기서도 별거 아닐 줄 알았는데, 한참 해멨다.

중간에 서버를 띄우고 API 요청을 날렸는데 403에러(API가 비활성화 되어있습니다. 어쩌구저쩌구,,)가 나와 IAM 권한만 생성했다 지웠다가, API 활성화 되어있는데 뭐지 하면서 헤맸다.
나중에 알고보니 client를 생성할 때 **GCP project id** 값을 넣어야 하는데, project name을 넣어서 작동하지 않는 에러였다. -_-...


#### API 호출 결과

![](https://www.dropbox.com/s/xa7ek6i4gqexbip/server_result.png?raw=1)

잘 돌아간다...! (속도는 매우 느리다...)

#### API monitoring
모델 대시보드에서 다음과 같이 API 호출에 대한 모니터링도 제공하고 있다.

![](https://www.dropbox.com/s/q739jusdmae6hzk/monitoring.png?raw=1)


## 느낀점
- 학습 시간이 오래 걸리는 모델을 학습 시킬 때나, batch prediction이 필요한 작업에서 파이프라인을 구성해서 유용하게 써볼 수 있을 것 같다. 제공하고 있는 monitoring, parameter tuning 등에 대해서도 좀 더 알아봐야 할 것 같다.
- 문서에서 명확하게 설명하지 않고 대충 스펙을 생략한 경우가 있어 생각보다 많이 헤맷다...
- 아직 shell script로만 제공하고 있는데, GUI로 된 인터페이스가 있다면 매우 유용해질 것 같다.
