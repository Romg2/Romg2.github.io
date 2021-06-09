---
title: "[Python] 나도코딩 웹 스크래핑편 - (14) 셀레니움4"
excerpt: "headless 셀레니움 실행하기"
categories: 
  - 나도코딩
tags: Python, 나도코딩
# toc: TRUE
# toc_sticky: TRUE
use_math: TRUE
date: 2021-06-09 18:40:00
---

유튜브 [나도코딩 웹 스크래핑](https://www.youtube.com/watch?v=yQ20jZwDjTE&t=17499s) 무료 강의를 통해 학습한 내용을 정리해서 올리고 있습니다.

실습과정에서 필요에 따라 일부 강의 내용의 누락 및 추가, 수정사항이 있습니다.

퀴즈의 경우, 유튜브 풀이와 상이할 수 있습니다.

---


**headless 셀레니움**

앞서 셀레니움 관련 포스팅에선 셀레니움을 이용해서 웹 정보를 가져왔다.

그동안 셀레니움을 실행하면 브라우저가 실제로 실행되고 명령어대로 조작되었다. 

이번엔 headless 옵션을 사용하여 브라우저를 실행하지 않고 같은 기능을 수행해보자.


```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.headless = True
options.add_argument("window-size=2560x1600") # 창 크기 변경
```

- headless를 True로 설정하여 브라우저를 실행하지 않고 셀레니움을 적용하게끔 옵션을 설정하였다.


- 나는 노트북을 써서 보이는 화면 크기가 작아 창 크기도 증가시키는 옵션을 설정하였다.


```python
import time
from bs4  import BeautifulSoup

browser = webdriver.Chrome("./chromedriver.exe", options=options)
browser.maximize_window() # 창 최대화

url = "https://play.google.com/store/movies/top"
headers = {
    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.104 Safari/537.36",
    "Accept-Language":"ko-KR,ko"
}


# 페이지 이동
browser.get(url)

# 반복해서 가장 아래로 스크롤 내리기
interval = 2
prev_height = browser.execute_script("return document.body.scrollHeight")

while True:
    # 화면 가장 아래로 스크롤 내리기
    browser.execute_script("window.scrollTo(0, document.body.scrollHeight)")

    # 페이지 로딩 대기
    time.sleep(interval)

    # 현재 문서 높이를 가져와서 저장
    curr_height = browser.execute_script("return document.body.scrollHeight")
    if curr_height == prev_height:
        break

    prev_height = curr_height
print("스크롤 완료")
```

    스크롤 완료
    


```python
# 잘 실행 되었는지 스크린샷
browser.get_screenshot_as_file("C:/Users/ekzm3/Desktop/Github_kkd/Github_blog/assets/images/post_images/2021-06-09-nado_web_14/google_movie.png")

soup = BeautifulSoup(browser.page_source, "lxml")

# 클래스를 2개 이상 가져올 때
# movies = soup.find_all("div", attrs= {"class":["ImZGtf mpg5gc","Vpfmgd"]})
movies = soup.find_all("div", attrs= {"class":"Vpfmgd"})

for i, movie in enumerate(movies):
    title = movie.find("div", attrs={"class":"WsMG1c nnK0zc"}).get_text()

    # 할인 전 가격
    original_price = movie.find("span", attrs={"class":"SUZt4c djCuy"})

    # 할인 후 가격
    discount_price = movie.find("span", attrs={"class": "VfPpfd ZdBevf i5DZme"}).get_text()

    if original_price:
        original_price = original_price.get_text()
    else:
        original_price = discount_price


    # 링크
    link = movie.find("a", attrs={"class":"JC71ub"})["href"]

    print(f"제목 : {title}")
    print(f"할인 전 금액 : {original_price}")
    print(f"할인 후 금액 : {discount_price}")
    print(f"링크 : https://play.google.com{link}")
    print("-" * 120)
    
    if i==4:
        break

browser.quit()
```

    제목 : 노바디
    할인 전 금액 : ₩1,800
    할인 후 금액 : ₩1,800
    링크 : https://play.google.com/store/movies/details/%EB%85%B8%EB%B0%94%EB%94%94?id=Bh7FPgXm7Ng.P
    ------------------------------------------------------------------------------------------------------------------------
    제목 : 고질라 VS. 콩
    할인 전 금액 : ₩5,000
    할인 후 금액 : ₩5,000
    링크 : https://play.google.com/store/movies/details/%EA%B3%A0%EC%A7%88%EB%9D%BC_VS_%EC%BD%A9?id=bKfIMqA5r6Q.P
    ------------------------------------------------------------------------------------------------------------------------
    제목 : 몬스터 헌터 Monster Hunter
    할인 전 금액 : ₩1,800
    할인 후 금액 : ₩1,800
    링크 : https://play.google.com/store/movies/details/%EB%AA%AC%EC%8A%A4%ED%84%B0_%ED%97%8C%ED%84%B0_Monster_Hunter?id=jdFtPHzGhVw.P
    ------------------------------------------------------------------------------------------------------------------------
    제목 : 콰이어트 플레이스 (자막판)
    할인 전 금액 : ₩1,000
    할인 후 금액 : ₩1,000
    링크 : https://play.google.com/store/movies/details/%EC%BD%B0%EC%9D%B4%EC%96%B4%ED%8A%B8_%ED%94%8C%EB%A0%88%EC%9D%B4%EC%8A%A4_%EC%9E%90%EB%A7%89%ED%8C%90?id=6uPeUO-J7Fs
    ------------------------------------------------------------------------------------------------------------------------
    제목 : 라야와 마지막 드래곤
    할인 전 금액 : ₩4,400
    할인 후 금액 : ₩4,400
    링크 : https://play.google.com/store/movies/details/%EB%9D%BC%EC%95%BC%EC%99%80_%EB%A7%88%EC%A7%80%EB%A7%89_%EB%93%9C%EB%9E%98%EA%B3%A4?id=mOj6B2R2qfo.P
    ------------------------------------------------------------------------------------------------------------------------
    

- 코드는 [구글 플레이 영화 정보 불러오기](https://romg2.github.io/%EB%82%98%EB%8F%84%EC%BD%94%EB%94%A9/nado_web_13_%EC%85%80%EB%A0%88%EB%8B%88%EC%9B%80_%EA%B5%AC%EA%B8%80-%EC%98%81%ED%99%94%EC%A0%95%EB%B3%B4/)에서 사용한 코드 그대로이다.


- headless 옵션을 적용하여 브라우저 실행 없이 똑같은 결과로 출력되는 것을 확인 할 수 있다.


```python
from IPython.display import Image

Image("C:/Users/ekzm3/Desktop/Github_kkd/Github_blog/assets/images/post_images/2021-06-09-nado_web_14/google_movie.png")
```




    
![output_7_0](https://user-images.githubusercontent.com/82348484/121332948-4ad6ed00-c953-11eb-9d58-a6d362d211c0.png)



- 브라우저 실행없이도 정말 똑같이 적용되었는지 앞서 `get_screenshot_as_file()`로 스크린샷을 저장하였다.


- 실제로 직접 홈페이지 들어가서 마지막까지 스크롤 했을 때와 같음을 확인 할 수 있다.

**headless 주의점**


```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.headless = True
options.add_argument("window-size=2560x1600")

browser = webdriver.Chrome("./chromedriver.exe", options=options)
browser.maximize_window() # 창 최대화

url = "https://www.whatismybrowser.com/detect/what-is-my-user-agent"
browser.get(url)

detected_value = browser.find_element_by_id("detected_value")
print(detected_value.text)
browser.quit()
```

    Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/91.0.4472.77 Safari/537.36
    

- 위 코드는 headless 옵션을 사용하여 브라우저 실행없이 User-Agent를 확인하는 사이트에 접속 후 User-Agent정보를 출력한다.


- 그런데 출력된 결과를 보면 User-Agent에 HeadlessChrome이라는 문자가 있다.


- 만약 특정 웹에 접근하는데 User-Agent를 확인할 때 headless가 붙어 있으면 접근을 거부 당할 수 있다.


```python
options.add_argument("user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36")
```

- headless 방식으로 웹 스크래핑 할 때 이같은 문제를 해결하려면 option에 user-agent 정보를 추가해주면 된다.


- 셀레니움을 더 알고 싶으면 [Selenium With Python](https://selenium-python.readthedocs.io/) 사이트를 참조하자.
