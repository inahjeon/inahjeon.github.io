---
layout: post
title:  "Selenium으로 Google Play Store 크롤링하기"
date:   2019-02-11 01:35:00
tags: [selenium, google play store, crawling]
comments: true
---

`Selenium`([https://www.seleniumhq.org](https://www.seleniumhq.org)) 으로 Google Play Store를 크롤링하는 간단한 웹 크롤러를 만들어 보았다.

## `Selenum` 이란?
브라우저를 자동화하는 툴로, 주로 자동화된 웹 테스팅, 웹 자동화 작업에 사용되는 라이브러리다. 데이터 분석 분야에서는 데이터를 크롤링하기 위한 용도로 많이 활용되고 있다.

**지원하는 웹 브라우저들**
- Firefox
- Internet Explorer
- Safari
- Opera
- Chrome

많이 사용되는 웬만한 브라우저는 다 지원하고 있고, 언어도 `Python`, `Java`, `R`, `Ruby` 등 널리 쓰이는 다양한 프로그래밍 언어를 지원하고 있다.


## 필요한 라이브러리 준비, 실행

크롤링을 위해 selenium 라이브러리를 설치하고, 사용할 웹 드라이버를 받아야한다.

### Selenium (Python)
```
pip install selenium
```

### Web driver

웹 드라이버는 [https://www.seleniumhq.org/about/platforms.jsp#browsers](https://www.seleniumhq.org/about/platforms.jsp#browsers) 에서 원하는 드라이버를 찾아서 다운받을 수 있는 링크로 이동할 수 있다. 여기선 익숙한 Chrome 드라이버를 설치해보았다.

![](https://user-images.githubusercontent.com/16538186/61559760-66a62100-aaa5-11e9-8780-fcdb0db3f3ca.png)

## 웹 브라우저 실행하기

일단 간단하게 웹 브라우저를 실행해보았다.

```python
from selenium import webdriver

driver = webdriver.Chrome('./chromedriver')
driver.get('https://www.google.com')
```

![](https://user-images.githubusercontent.com/16538186/61559764-69a11180-aaa5-11e9-901a-83d9ff3b1538.png)

코드를 실행하면 웹 브라우저가 뜨고 입력한 url로 이동한다.

## Play Store 인기차트 크롤링

Selenium으로 어떤 데이터를 크롤링 해볼까하다가 Google Play Store의 인기 앱 차트 목록을 간단히 크롤링 하는 작업을 해보기로 했다.

![](https://user-images.githubusercontent.com/16538186/61559772-6dcd2f00-aaa5-11e9-875d-34faf676075e.png)

(뱅크샐러드가 인기 앱 11위에요! 😎 ~~깨알같이 자랑...~~)

### 크롤링 하고 싶은 영역 찾기

`html` 문서를 크롤링 하기 위해서는 원하는 html element를 선택해서 내용을 가져와야 한다. Chrome 브라우저의 **개발자도구**를 활용해서 원하는 내용을 포함하고 있는 element를 선택하기 위한 정보(`id`, `class name`, `tag`... 등을 확인할 수 있다.

![](https://user-images.githubusercontent.com/16538186/61559777-7160b600-aaa5-11e9-93df-5198761a3208.png)

`class name` 이 details 인 `div` 안에 원하는 내용이 있어서 `find_elements_by_class_name` 함수를 활용해서 내용을 가져올 수 있었다.

```python
from selenium import webdriver

driver = webdriver.Chrome('./chromedriver')
driver.get('https://play.google.com/store/apps/collection/topselling_free')
top_app_details = driver.find_elements_by_class_name('details')

for app in top_app_details:
    print(app.text)
```

실행 결과: 

```
1. 브롤스타즈
Supercell
2. Color Bump 3D
Good Job Games
3. YouTube Kids
Google LLC
4. CGV
CJ CGV
5. Netflix(넷플릭스)
Netflix, Inc.
6. 카카오톡 KakaoTalk
Kakao Corporation
7. Samsung Smart Switch Mobile
Samsung Electronics Co., Ltd.
8. 신비아파트 고스트헌터
주식회사 3F FACTORY
...
```

내용을 잘 긁어온 것을 확인할 수 있다.

### 카테고리 별, 유료/무료 인기 top N개의 앱 리스트를 긁어오기

```python
from selenium import webdriver
from datetime import datetime


def crawling_top_apps(
    web_driver,
    category: str = '',
    paid: str = '',
    limit: int = 10
):
    category_url = f'/category/{category}'
    free_app_rank_url = f'https://play.google.com/store/apps{category_url}' \
                        f'/collection/topselling_{paid}'

    web_driver.get(free_app_rank_url)
    top_app_details = web_driver.find_elements_by_class_name('details')

    for idx, app in enumerate(top_app_details[:limit]):
        content = app.text.split('. ')[-1]

        app_title = content.split('\n')[0]
        corp = content.split('\n')[-1]

        print(f'{idx + 1}위 {app_title} - {corp}')


if __name__ == '__main__':
    top_n = 10
    categories = [
        'HEALTH_AND_FITNESS',
        'FINANCE',
        'SHOPPING'
    ]
    paid = 'free'

    driver = webdriver.Chrome('./chromedriver')

    today = datetime.now().strftime('%Y-%m-%d')

    for category in categories:
        print('-' * 60)
        print(f'{today} Top {top_n} {paid} Apps in {category}')
        print('-' * 60)
        crawling_top_apps(driver, category, paid, top_n)
```

실행 결과:

```
------------------------------------------------------------
2019-02-11 Top 10 free Apps in HEALTH_AND_FITNESS
------------------------------------------------------------
1위 Samsung Health(삼성 헬스) - Samsung Electronics Co., Ltd.
2위 캐시슬라이드 스텝업 - No.1 적립형 만보기앱 - NBT Inc.
3위 M건강보험 - 국민건강보험공단
4위 AIA Vitality x T건강걷기 - AIA Vitality (바이탈리티)
5위 복근 운동 - 28일 이내 지방을 연소시켜 111자 복근을 만들어줍니다 - Super T Group
6위 마이아큐브 - (주)한국존슨앤드존슨
7위 ANT+ Plugins Service - ANT+
8위 홈 트레이닝 - 기구가 필요 없습니다 - Leap Fitness Group
9위 만보기 - ITO Technologies, Inc.
10위 건강iN(건강인) - 국민건강보험공단
------------------------------------------------------------
2019-02-11 Top 10 free Apps in FINANCE
------------------------------------------------------------
1위 뱅크샐러드 - 신경 꺼도 내 돈 관리 (통합 자산관리, 자동 가계부) - Rainist Co., Ltd.
2위 NH스마트뱅킹 - 농협BANK
3위 카카오뱅크 - 같지만 다른 은행 - 카카오뱅크
4위 토스 - Viva Republica
5위 ISP/페이북 - VP Inc
6위 KB국민은행 스타뱅킹 - KB국민은행
7위 NH콕뱅크 - 농협BANK
8위 터치엔 엠백신 for Web(기업용) - 라온시큐어(주)
9위 KB국민카드 - KB KOOKMINCARD CO., LTD.
10위 바이오인증 공동앱 - 금융결제원(KFTC)
------------------------------------------------------------
2019-02-11 Top 10 free Apps in SHOPPING
------------------------------------------------------------
1위 쿠팡 (Coupang) - 쿠팡
2위 당근마켓 - 우리 동네 중고 직거래 벼룩장터 - 당근마켓
3위 롯데홈쇼핑 LOTTE Homeshopping - Lotte Homeshopping
4위 위메프 - 특가대표 (특가 / 쇼핑 / 쇼핑앱 / 쿠폰 / 배송) - 위메프
5위 SSG.COM - SSG.COM
6위 공구마켓 - 제이슨 컨텐츠
7위 현대Hmall - 홈쇼핑, 백화점 - (주) 현대홈쇼핑
8위 롯데백화점 - 롯데쇼핑
9위 심쿵할인 - 제이슨 컨텐츠
10위 GS SHOP - 당신의 가장 좋은 선택을 만듭니다 - GS SHOP
```

완성! (~~금융 앱 1위는 역시 뱅크샐러드~~ 💰)


자, 그럼 본격적으로 앱 정보, 리뷰 데이터까지 다 긁어오는 기능까지 만들자! 

*라고 생각했지만... 크롤링 코드를 짜면서 html 요소를 찾고 값을 가져오고 하는 작업이 생각보다 귀찮았고 (-.-) 찾아보니 당연하게도 Play store 데이터를 크롤링하는 오픈소스들이 많이 있어서 바로 생각을 접었다. ~~크롤링할 영역 클릭만 하면 알아서 크롤링해주는 오픈소스 누가 만들어 주면 좋겠다.~~*

### Google login

뭔가 아쉬워서 Google 계정 로그인을 하는 스크립트도 만들어보았다. 로그인을 하기 위해서는 input form에 계정정보를 입력하고, 로그인 버튼을 클릭하는 동작을 지정해야한다. input form의 값은 `send_keys`라는 함수로 넣을 수 있고, `click` 함수를 사용해서 html element 요소를 click하는 액션을 지정할 수 있다.

![](https://user-images.githubusercontent.com/16538186/61559784-745ba680-aaa5-11e9-94a3-2fc16df528cc.png)
![](https://user-images.githubusercontent.com/16538186/61559790-79205a80-aaa5-11e9-8bfd-471d2c496539.png)

**Google 계정 로그인 프로세스**는 

1) email 주소를 입력하고

2) 다음 버튼을 클릭

3) password 를 입력하고

4) 다음 버튼을 클릭

의 단계로 진행된다.

```python
from selenium import webdriver
from meta import MY_EMAIL, MY_PASSWORD


if __name__ == '__main__':
    driver = webdriver.Chrome('./chromedriver')

    driver.implicitly_wait(3)

    driver.get('https://accounts.google.com/ServiceLogin?hl=ko&passive=true&continue=https://www.google.com/')

    driver.find_element_by_name('identifier').send_keys(MY_EMAIL)

    driver.find_element_by_id('identifierNext').click()

    driver.find_element_by_name('password').send_keys(MY_PASSWORD)
    
    element = driver.find_element_by_id('passwordNext')
    driver.execute_script("arguments[0].click();", element)
```

로그인 프로세스대로 동작을 넣고, 실행해보면 알아서 값을 넣고 로그인하는 걸 볼 수 있다.

컴퓨터가 혼자서 내 계정에 로그인 하는걸 보고 있으면 뭔가 기분이 묘하다;

