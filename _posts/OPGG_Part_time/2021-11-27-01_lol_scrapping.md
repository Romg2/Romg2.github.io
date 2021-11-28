---
title: "[OPGG] 인턴 연계 과정 - 프로 리그 데이터 수집"
excerpt: "인턴 연계 과정 공부 내용"
categories: 
  - OPGG_Part
tags: 
    - Python
    - OPGG_Part
toc: TRUE
toc_sticky: TRUE
use_math: TRUE
---

# 오피지지 인턴 과정

오피지지 데이터 분석가 과정 교육을 수료하고 우수 교육생으로 한 달간 인턴 생활을 하게 되었습니다.

회사의 실무를 보면서 개인적으로 진행했던 과정에 대해 정리하고자 합니다.

본 내용들은 실제 인턴 생활에서 맡았던 업무와 관련 있다고 생각한 부분을 개인적으로 진행한 것 입니다.

실제 맡은 업무와는 상이할 수 있으니 참고 바랍니다.

## 프로 리그 데이터 수집

여기서는 스크래핑을 이용해서 리그 오브 레전드 프로 리그의 데이터를 수집하고자 합니다.

현재 개인이 Riot API를 이용해서 프로 리그 데이터를 가져올 수 없기 때문에 스크래핑을 사용했습니다.

### 1. 스크래핑 사이트

스크래핑 사이트로는 [QWER.GG](https://qwer.gg/)를 사용했습니다.

한국 뿐 아니라 여러 리그의 경기 정보가 담겨있는 사이트입니다.

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-27-01/qwergg.PNG?raw=true)

- 홈페이지 상단에 LCK, LPL등 다양한 리그가 있습니다.

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-27-01/lck.PNG?raw=true)

- LCK를 클릭했을 때 모습입니다.


- 연도별 롤드컵 결정전, 섬머, 스프링 등을 클릭하여 경기 결과 요약을 확인 할 수 있습니다.


- 사진에는 보이지 않지만 경기 결과 하단에는 `더보기` 버튼이 있어 이전 경기 결과도 확인 가능합니다.

![](https://github.com/Romg2/Romg2.github.io/blob/master/_posts/OPGG_Part_time/2021-11-27-01/result.PNG?raw=true)

- 경기를 클릭하면 다음과 같이 상세 정보가 나타납니다.


- 세트별 정보, 룬 등의 상세 정보는 다시 클릭을 통해 확인이 가능합니다.

### 2. 수집 방법

우선 사이트의 특성 상 클릭을 해야 정보가 생기는 부분이 많아 `selenium`을 사용해야 합니다.

이를 이용해서 각 경기 결과의 정보를 가져와 보겠습니다.

**[데이터 수집]**


```python
import numpy as np
import pandas as pd
from pandas.api.types import CategoricalDtype

import requests
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver import ActionChains

import time

import pytube

import warnings
warnings.filterwarnings("ignore")
```


```python
# 원하는 형태의 url 만들기
want_season = "LCK 2021 Summer"

url_sp = want_season.split(" ")
league = url_sp.pop(0)
year = url_sp.pop(0)

if (url_sp[-1] == "Summer") | (url_sp[-1] == "Spring") | (url_sp[-1] == "Winter"):
    season = url_sp[-1]
else:
    season = "/".join(url_sp)

url = f"https://qwer.gg/leagues/{league}/{year}/{season}"
print(url)
```

    https://qwer.gg/leagues/LCK/2021/Summer
    

- 우선 url의 형태는 특정 리그, 연도, 토너먼트를 입력하여야 합니다. 


- 예시: `https://qwer.gg/leagues/LCK/2021/summer`


- 여기서 롤드컵 결정전은 Regional-finals로 입력하여야 합니다.


- 사실 if문을 안써도 상관없지만 다른 작업과 병행하면서 예외 처리로 넣어둔 것이니 무시해도 됩니다.


```python
# 크롬 드라이버 옵션
options = webdriver.ChromeOptions()
options.add_argument('headless')
options.add_argument("--window-size=1920x1080")

# 크롬 드라이버 실행
browser = webdriver.Chrome("./chromedriver.exe", options= options)
browser.maximize_window()
browser.get(url)

# 더보기 클릭하여 모든 링크 찾기
while True:
    try:
        WebDriverWait(browser,5).until(
            EC.presence_of_element_located( 
                (By.XPATH, '//*[@id="root"]/div[3]/div[2]/div/div[4]/div/section[1]/section[1]/div[2]/div[2]/div/div[2]/button') )).click()
    except:
        break
        time.sleep(2)
        
# 링크 저장
soup = BeautifulSoup(browser.page_source, "lxml")
browser.quit()
soup2 = soup.find("div", attrs={"class":"Tournament__records"}).find("div", attrs={"class":"MatchItemList"})
```

- 사이트에 접속하였다면 `더보기` 버튼을 계속 클릭하여 모든 경기 결과 요약 정보를 확인해야합니다.


- 누르지 않는다면 해당 사이트에 다른 경기 정보는 없습니다.


- 더 이상 경기가 없을 때까지 버튼을 클릭 후 페이지 정보를 soup에 저장하고 브라우저를 종료합니다.


- 경기 결과 요약 정보가 있는 전체 테이블 정보를 soup2에 저장합니다.


- 왜 해당 태그들이나 클래스를 썼는지는 직접 확인하면서 작업합니다.


```python
# 경기별 url
url_lst = []
for i in soup2.find_all("a", attrs={"class":"MatchItem__link"}):
    url_lst.append("https://qwer.gg" + i["href"])
```


```python
print(f"총 매치 수: {len(url_lst)}")
```

    총 매치 수: 95
    


```python
url_lst[:5]
```




    ['https://qwer.gg/matches/XSDD-fIWa/2021-06-09-BRO-vs-LSB',
     'https://qwer.gg/matches/IsULOWisZC/2021-06-09-T1-vs-HLE',
     'https://qwer.gg/matches/DlJQeVq38J/2021-06-10-KT-vs-NS',
     'https://qwer.gg/matches/FQOaDQ--GT/2021-06-10-GEN-vs-DRX',
     'https://qwer.gg/matches/DbzcjTdnct/2021-06-11-AF-vs-BRO']



- LCK 2021 Summer 시즌에는 총 95개의 매치(set 아님)가 있었습니다.


- 가장 첫 번째 경기는 프레딧 브리온과 리브샌드박스의 경기네요.


- 각 경기 결과의 상세 정보 url은 url_lst에 저장하였습니다.


```python
def match_url(want_season):
    # 원하는 형태의 url 만들기
    url_sp = want_season.split(" ")
    league = url_sp.pop(0)
    year = url_sp.pop(0)

    if (url_sp[-1] == "Summer") | (url_sp[-1] == "Spring") | (url_sp[-1] == "Winter"):
        season = url_sp[-1]
    else:
        season = "/".join(url_sp)

    url = f"https://qwer.gg/leagues/{league}/{year}/{season}"

    # 크롬 드라이버 옵션
    options = webdriver.ChromeOptions()
    options.add_argument('headless')
    options.add_argument("--window-size=1920x1080")

    # 크롬 드라이버 실행
    browser = webdriver.Chrome("./chromedriver.exe", options= options)
    browser.maximize_window()
    browser.get(url)

    # 더보기 클릭하여 모든 링크 찾기
    while True:
        try:
            WebDriverWait(browser,5).until(
                EC.presence_of_element_located( 
                    (By.XPATH, '//*[@id="root"]/div[3]/div[2]/div/div[4]/div/section[1]/section[1]/div[2]/div[2]/div/div[2]/button') )).click()
        except:
            break
            time.sleep(2)

    # 링크 저장
    soup = BeautifulSoup(browser.page_source, "lxml")
    browser.quit()
    soup2 = soup.find("div", attrs={"class":"Tournament__records"}).find("div", attrs={"class":"MatchItemList"})

    # 경기별 url
    url_lst = []
    for i in soup2.find_all("a", attrs={"class":"MatchItem__link"}):
        url_lst.append("https://qwer.gg" + i["href"])
        
    return url_lst
```


```python
url_lst = match_url(want_season="LCK 2021 summer")
```

- 좀 전까지의 모든 과정을 하나의 함수로 정의합니다.


```python
# 경기 상세 정보 url
url = url_lst[0]

# 크롬 드라이버 옵션
options = webdriver.ChromeOptions()
options.add_argument('headless')
options.add_argument("--window-size=1920x1080")

# 크롬 드라이버 실행
browser = webdriver.Chrome("./chromedriver.exe", options= options)
browser.get(url)
browser.maximize_window()

# 채팅 닫기
browser.find_element_by_css_selector('#root > div.Chat.Chat--opened > div.Chat__window > div.Chat__header > svg').click()

# 빈 데이터 프레임
final_data = pd.DataFrame()

# 세트 선택
set_num = 1

while True:
    try:
        # 세트 클릭
        WebDriverWait(browser,3).until(
            EC.presence_of_element_located( 
                (By.XPATH, f"//*[contains(text(), 'SET {set_num}')]") )).click()

        time.sleep(2)

        # 챔피언 기본 정보
        soup3 = BeautifulSoup(browser.page_source, "lxml")
        temp = soup3.find("div", attrs={"class":"Tabs__contents"}).find_all("section")[0]

        data = pd.DataFrame()
        for s in range(10):
            info_lst = []

            # 챔피언
            info_lst.append(temp.find_all("li")[s].find("img")["alt"])
            # 레벨
            info_lst.append(temp.find_all("li")[s].find("span").get_text())
            # 스펠
            info_lst.append(temp.find_all("li")[s].find_all("img", attrs={"class":"SpellIcon__image"})[0]["alt"])
            info_lst.append(temp.find_all("li")[s].find_all("img", attrs={"class":"SpellIcon__image"})[1]["alt"])
            # 선수 이름
            info_lst.append(temp.find_all("li")[s].find("a", attrs={"class":"MatchHistoryDetailPlayer__summoner__link"}).get_text())
            # kda
            kda = temp.find_all("li")[s].find("div", attrs={"class":"MatchHistoryDetailPlayer__kda"}).find_all("span")[0].get_text()
            k,d,a = kda.split("/")
            info_lst.append(k), info_lst.append(d), info_lst.append(a)

            # 가한 데미지
            info_lst.append(temp.find_all("li")[s].find("span", attrs={"class":"MatchHistoryDetailPlayer__damage__desktop"}).get_text().replace(",",""))
            # 시야점수
            info_lst.append(temp.find_all("li")[s].find("div", attrs={"class":"MatchHistoryDetailPlayer__vs"}).get_text())
            # CS
            cs_lst = temp.find_all("li")[s].find("div", attrs={"class":"MatchHistoryDetailPlayer__cs"})["title"].split()
            mcs = int(cs_lst[1])
            jcs = int(cs_lst[3])
            cs = mcs + jcs
            info_lst.append(mcs), info_lst.append(jcs), info_lst.append(cs)

            # 아이템
            for i, info in enumerate(temp.find_all("li")[s].find_all("div", attrs={"class":"ItemIcon MatchHistoryDetailPlayer__items__elem"})):
                try:
                    info_lst.append(info.find('img')['alt'])
                except:
                    info_lst.append("None")

            # 장신구
            info_lst.append(temp.find_all("li")[s].find("div", attrs={"class":"ItemIcon MatchHistoryDetailPlayer__items__ward"}).find("img")["alt"])

            # 데이터 프레임화
            data = pd.concat([data, pd.DataFrame(info_lst).T], axis=0)


        # 밴
        ban_lst = []
        for i in range(2):
            ban_info = soup3.find_all("div", attrs={"class":"MatchHistoryDetailBansAndObjects__ban"})[i].find_all("a")
            for j in range(5):
                try:
                    ban_lst.append(ban_info[j].find("img")["alt"])
                except:
                    ban_lst.append("None")
        data["ban"] = ban_lst

        # 진영
        data["side"] = ["blue"]*5 + ["red"]*5

        # 결과
        blue_result = temp.find_all("div", attrs={"class":"MatchHistoryDetailOverall__player"})[0].get_text().split(" ")[0]
        red_result = temp.find_all("div", attrs={"class":"MatchHistoryDetailOverall__player"})[1].get_text().split(" ")[0]

        data["result"] = [blue_result]*5 + [red_result]*5

        # 오브젝트
        temp21 = temp.find_all("div", attrs={"class":"MatchHistoryDetailBansAndObjects__object"})[0]
        temp31 = temp21.find_all("span", attrs={"class":"ObjectIcon MatchHistoryDetailBansAndObjects__object__icon"})
        temp22 = temp.find_all("div", attrs={"class":"MatchHistoryDetailBansAndObjects__object"})[3]
        temp32 = temp22.find_all("span", attrs={"class":"ObjectIcon MatchHistoryDetailBansAndObjects__object__icon"})

        data["turret"] = [temp31[0].get_text()]*5 + [temp32[0].get_text()]*5
        data["inhibitor"] = [temp31[1].get_text()]*5 + [temp32[1].get_text()]*5
        data["baron"] = [temp31[2].get_text()]*5 + [temp32[2].get_text()]*5
        data["dragon"] = [temp31[3].get_text()]*5 + [temp32[3].get_text()]*5
        data["riftHerald"] = [temp31[4].get_text()]*5 + [temp32[4].get_text()]*5

        # 경기 시간, 날짜, 버전, url
        length_lst = temp.find("div", attrs={"class":"MatchHistoryDetail__duration"}).get_text().split(":")
        data["length"] = int(length_lst[0])*60 + int(length_lst[1])
        data["date"] = temp.find("div", attrs={"class":"GameDetail__date"}).get_text().split(" ")[0].replace(".","-")
        data["version"] = temp.find("div", attrs={"class":"GameDetail__date"}).get_text().split(" ")[-1]

        # 룬 정보 클릭
        WebDriverWait(browser,5).until(
            EC.presence_of_element_located( 
                (By.XPATH, f"//*[contains(text(), '룬 / 빌드')]") )).click()
        time.sleep(1.5)

        rune_df = pd.DataFrame()

        # 챔피언별 룬 페이지 클릭
        for i in ['blue', 'red']:
            for j in range(1,6):
                rune_stat_lst = []
                WebDriverWait(browser, 5).until(
                    EC.presence_of_element_located( 
                        (By.CSS_SELECTOR, f'#root > div.Match > div:nth-child(7) > div > div > div.Tabs__contents > section > div.MatchHistoryDetail.GameDetail__matchHistory > section:nth-child(4) > div > div.Tabs__contents > section > div > div.MatchHistoryDetailBuilds__teams > div.MatchHistoryDetailBuilds__teams__{i} > div > div:nth-child({j}) > a > div > img') )).click()
                time.sleep(1)

                soup2 = BeautifulSoup(browser.page_source, "lxml")

                # 메인룬, 서브룬
                rune_stat_lst.append(soup2.find_all("span", attrs={"class":"RuneTree__styleName"})[0].get_text())
                rune_stat_lst.append(soup2.find_all("span", attrs={"class":"RuneTree__styleName"})[1].get_text())

                # 세부 룬 정보
                for k in soup2.find_all("div", attrs={"class":"RuneTree__rune RuneIcon"}):
                    rune_stat_lst.append(k.find("img")["alt"])

                for k in soup2.find_all("div", attrs={"class":"StatPerkIcon RuneTree__perk"}):
                    rune_stat_lst.append(k.find("img")["alt"])

                rune_df = pd.concat([rune_df, pd.DataFrame(rune_stat_lst).T], axis=0)

        # 기본 정보, 룬 정보 결합
        set_data = pd.concat([data, rune_df], axis=1)

        # 최종 데이터
        final_data = pd.concat([final_data, set_data], axis=0)
        set_num += 1
    except:
        browser.find_elements_by_xpath("//*[contains(text(), '전체경기')]")[0].click()
        soup4 = BeautifulSoup(browser.page_source, "lxml")
        final_data["url"] = soup4.find("iframe", attrs={"class":"YoutubeVideo"})["src"]
        browser.quit()
        break
```

- 경기 상세 정보 url에 접속했다면 팀별 챔피언, 밴 정보 등의 태그를 확인 후 수집합니다.


- 이 과정에서 접속시 팝업창 같은 채팅창을 종료하는 코드를 추가했습니다.


- 룬 페이지를 클릭하여 선수별 룬 정보도 수집합니다.


- 이 과정을 모든 세트에 대해 반복하며 보통 2~5개 존재할 수 있으므로 `while`을 이용하여 작업합니다.


- 직접 작업을 하면서 로딩 시간 등을 고려하여 적절히 `sleep`등을 활용해야 합니다.


- 주의 사항으로는 밴의 경우 2021 기준 10개가 맞지만 징계 등으로 인해 밴이 없는 경우가 존재 합니다.


- 또한 아이템도 6개 + 1개(장신구)가 항상 모두 있진 않을 것입니다.


- 데이터 프레임으로 만드면서 이러한 예외 처리를 잘 해주어야 합니다.


```python
print(final_data.shape)
```

    (20, 43)
    

- 데이터는 잘 20 x 43 형태로 구축되었습니다.


```python
final_data.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>나르</td>
      <td>18</td>
      <td>점멸</td>
      <td>순간이동</td>
      <td>BRO Hoya</td>
      <td>2</td>
      <td>0</td>
      <td>9</td>
      <td>19967</td>
      <td>46</td>
      <td>...</td>
      <td>착취의 손아귀</td>
      <td>철거</td>
      <td>뼈 방패</td>
      <td>과잉성장</td>
      <td>피의 맛</td>
      <td>굶주린 사냥꾼</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>릴리아</td>
      <td>16</td>
      <td>강타</td>
      <td>점멸</td>
      <td>BRO UmTi</td>
      <td>2</td>
      <td>3</td>
      <td>13</td>
      <td>11555</td>
      <td>55</td>
      <td>...</td>
      <td>난입</td>
      <td>빛의 망토</td>
      <td>기민함</td>
      <td>물 위를 걷는 자</td>
      <td>외상</td>
      <td>우주적 통찰력</td>
      <td>5008</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>루시안</td>
      <td>18</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>BRO Lava</td>
      <td>11</td>
      <td>0</td>
      <td>5</td>
      <td>22265</td>
      <td>71</td>
      <td>...</td>
      <td>집중 공격</td>
      <td>침착</td>
      <td>전설: 핏빛 길</td>
      <td>최후의 저항</td>
      <td>비스킷 배달</td>
      <td>시간 왜곡 물약</td>
      <td>5005</td>
      <td>5008</td>
      <td>5003</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>칼리스타</td>
      <td>17</td>
      <td>회복</td>
      <td>점멸</td>
      <td>BRO Hena</td>
      <td>3</td>
      <td>1</td>
      <td>10</td>
      <td>9735</td>
      <td>87</td>
      <td>...</td>
      <td>칼날비</td>
      <td>피의 맛</td>
      <td>좀비 와드</td>
      <td>굶주린 사냥꾼</td>
      <td>비스킷 배달</td>
      <td>우주적 통찰력</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>그라가스</td>
      <td>14</td>
      <td>점화</td>
      <td>점멸</td>
      <td>BRO Delight</td>
      <td>1</td>
      <td>2</td>
      <td>11</td>
      <td>5349</td>
      <td>111</td>
      <td>...</td>
      <td>여진</td>
      <td>생명의 샘</td>
      <td>재생의 바람</td>
      <td>불굴의 의지</td>
      <td>비스킷 배달</td>
      <td>우주적 통찰력</td>
      <td>5007</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>리 신</td>
      <td>18</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>LSB Summit</td>
      <td>3</td>
      <td>2</td>
      <td>1</td>
      <td>13179</td>
      <td>36</td>
      <td>...</td>
      <td>정복자</td>
      <td>승전보</td>
      <td>전설: 강인함</td>
      <td>최후의 저항</td>
      <td>재생의 바람</td>
      <td>소생</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>올라프</td>
      <td>14</td>
      <td>강타</td>
      <td>점멸</td>
      <td>LSB Croco</td>
      <td>1</td>
      <td>6</td>
      <td>4</td>
      <td>8487</td>
      <td>63</td>
      <td>...</td>
      <td>정복자</td>
      <td>승전보</td>
      <td>전설: 민첩함</td>
      <td>최후의 일격</td>
      <td>마법의 신발</td>
      <td>쾌속 접근</td>
      <td>5008</td>
      <td>5008</td>
      <td>5003</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>빅토르</td>
      <td>17</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>LSB FATE</td>
      <td>2</td>
      <td>4</td>
      <td>3</td>
      <td>19753</td>
      <td>32</td>
      <td>...</td>
      <td>콩콩이 소환</td>
      <td>마나순환 팔찌</td>
      <td>절대 집중</td>
      <td>주문 작열</td>
      <td>비스킷 배달</td>
      <td>시간 왜곡 물약</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>진</td>
      <td>15</td>
      <td>회복</td>
      <td>점멸</td>
      <td>LSB Prince</td>
      <td>0</td>
      <td>2</td>
      <td>3</td>
      <td>10324</td>
      <td>28</td>
      <td>...</td>
      <td>기민한 발놀림</td>
      <td>침착</td>
      <td>전설: 핏빛 길</td>
      <td>최후의 일격</td>
      <td>비스킷 배달</td>
      <td>시간 왜곡 물약</td>
      <td>5008</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>카르마</td>
      <td>11</td>
      <td>점멸</td>
      <td>탈진</td>
      <td>LSB Effort</td>
      <td>0</td>
      <td>5</td>
      <td>4</td>
      <td>6655</td>
      <td>111</td>
      <td>...</td>
      <td>신비로운 유성</td>
      <td>빛의 망토</td>
      <td>절대 집중</td>
      <td>주문 작열</td>
      <td>비스킷 배달</td>
      <td>우주적 통찰력</td>
      <td>5008</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 43 columns</p>
</div>



- 프레딧 브리온과 리브 샌드박스의 경기 결과 정보입니다.


- 1세트 당 10개의 챔피언이 있고 스펠, 아이템, KDA, 경기 영상 url등이 존재하나 컬럼명이 구분 되지 않습니다.


- 처음 확인했을 때 20개의 row가 있었으니 두 팀의 결과는 2세트로 2:0 이었겠네요.


```python
def league_data2(url):
    # 크롬 드라이버 옵션
    options = webdriver.ChromeOptions()
    options.add_argument('headless')
    options.add_argument("--window-size=1920x1080")

    # 크롬 드라이버 실행
    browser = webdriver.Chrome("./chromedriver.exe", options= options)
    browser.get(url)
    browser.maximize_window()

    # 채팅 닫기
    browser.find_element_by_css_selector('#root > div.Chat.Chat--opened > div.Chat__window > div.Chat__header > svg').click()

    # 빈 데이터 프레임
    final_data = pd.DataFrame()

    # 세트 선택
    set_num = 1

    while True:
        try:
            # 세트 클릭
            WebDriverWait(browser,3).until(
                EC.presence_of_element_located( 
                    (By.XPATH, f"//*[contains(text(), 'SET {set_num}')]") )).click()
            
            time.sleep(2)

            # 챔피언 기본 정보
            soup3 = BeautifulSoup(browser.page_source, "lxml")
            temp = soup3.find("div", attrs={"class":"Tabs__contents"}).find_all("section")[0]
            
            data = pd.DataFrame()
            for s in range(10):
                info_lst = []
                
                # 챔피언
                info_lst.append(temp.find_all("li")[s].find("img")["alt"])
                # 레벨
                info_lst.append(temp.find_all("li")[s].find("span").get_text())
                # 스펠
                info_lst.append(temp.find_all("li")[s].find_all("img", attrs={"class":"SpellIcon__image"})[0]["alt"])
                info_lst.append(temp.find_all("li")[s].find_all("img", attrs={"class":"SpellIcon__image"})[1]["alt"])
                # 선수 이름
                info_lst.append(temp.find_all("li")[s].find("a", attrs={"class":"MatchHistoryDetailPlayer__summoner__link"}).get_text())
                # kda
                kda = temp.find_all("li")[s].find("div", attrs={"class":"MatchHistoryDetailPlayer__kda"}).find_all("span")[0].get_text()
                k,d,a = kda.split("/")
                info_lst.append(k), info_lst.append(d), info_lst.append(a)
                
                # 가한 데미지
                info_lst.append(temp.find_all("li")[s].find("span", attrs={"class":"MatchHistoryDetailPlayer__damage__desktop"}).get_text().replace(",",""))
                # 시야점수
                info_lst.append(temp.find_all("li")[s].find("div", attrs={"class":"MatchHistoryDetailPlayer__vs"}).get_text())
                # CS
                cs_lst = temp.find_all("li")[s].find("div", attrs={"class":"MatchHistoryDetailPlayer__cs"})["title"].split()
                mcs = int(cs_lst[1])
                jcs = int(cs_lst[3])
                cs = mcs + jcs
                info_lst.append(mcs), info_lst.append(jcs), info_lst.append(cs)
                
                # 아이템
                for i, info in enumerate(temp.find_all("li")[s].find_all("div", attrs={"class":"ItemIcon MatchHistoryDetailPlayer__items__elem"})):
                    try:
                        info_lst.append(info.find('img')['alt'])
                    except:
                        info_lst.append("None")

                # 장신구
                info_lst.append(temp.find_all("li")[s].find("div", attrs={"class":"ItemIcon MatchHistoryDetailPlayer__items__ward"}).find("img")["alt"])
                
                # 데이터 프레임화
                data = pd.concat([data, pd.DataFrame(info_lst).T], axis=0)
                

            # 밴
            ban_lst = []
            for i in range(2):
                ban_info = soup3.find_all("div", attrs={"class":"MatchHistoryDetailBansAndObjects__ban"})[i].find_all("a")
                for j in range(5):
                    try:
                        ban_lst.append(ban_info[j].find("img")["alt"])
                    except:
                        ban_lst.append("None")
            data["ban"] = ban_lst

            # 진영
            data["side"] = ["blue"]*5 + ["red"]*5

            # 결과
            blue_result = temp.find_all("div", attrs={"class":"MatchHistoryDetailOverall__player"})[0].get_text().split(" ")[0]
            red_result = temp.find_all("div", attrs={"class":"MatchHistoryDetailOverall__player"})[1].get_text().split(" ")[0]

            data["result"] = [blue_result]*5 + [red_result]*5

            # 오브젝트
            temp21 = temp.find_all("div", attrs={"class":"MatchHistoryDetailBansAndObjects__object"})[0]
            temp31 = temp21.find_all("span", attrs={"class":"ObjectIcon MatchHistoryDetailBansAndObjects__object__icon"})
            temp22 = temp.find_all("div", attrs={"class":"MatchHistoryDetailBansAndObjects__object"})[3]
            temp32 = temp22.find_all("span", attrs={"class":"ObjectIcon MatchHistoryDetailBansAndObjects__object__icon"})

            data["turret"] = [temp31[0].get_text()]*5 + [temp32[0].get_text()]*5
            data["inhibitor"] = [temp31[1].get_text()]*5 + [temp32[1].get_text()]*5
            data["baron"] = [temp31[2].get_text()]*5 + [temp32[2].get_text()]*5
            data["dragon"] = [temp31[3].get_text()]*5 + [temp32[3].get_text()]*5
            data["riftHerald"] = [temp31[4].get_text()]*5 + [temp32[4].get_text()]*5

            # 경기 시간, 날짜, 버전, url
            length_lst = temp.find("div", attrs={"class":"MatchHistoryDetail__duration"}).get_text().split(":")
            data["length"] = int(length_lst[0])*60 + int(length_lst[1])
            data["date"] = temp.find("div", attrs={"class":"GameDetail__date"}).get_text().split(" ")[0].replace(".","-")
            data["version"] = temp.find("div", attrs={"class":"GameDetail__date"}).get_text().split(" ")[-1]

            # 룬 정보 클릭
            WebDriverWait(browser,5).until(
                EC.presence_of_element_located( 
                    (By.XPATH, f"//*[contains(text(), '룬 / 빌드')]") )).click()
            time.sleep(1.5)
            
            rune_df = pd.DataFrame()
            
            # 챔피언별 룬 페이지 클릭
            for i in ['blue', 'red']:
                for j in range(1,6):
                    rune_stat_lst = []
                    WebDriverWait(browser, 5).until(
                        EC.presence_of_element_located( 
                            (By.CSS_SELECTOR, f'#root > div.Match > div:nth-child(7) > div > div > div.Tabs__contents > section > div.MatchHistoryDetail.GameDetail__matchHistory > section:nth-child(4) > div > div.Tabs__contents > section > div > div.MatchHistoryDetailBuilds__teams > div.MatchHistoryDetailBuilds__teams__{i} > div > div:nth-child({j}) > a > div > img') )).click()
                    time.sleep(1)
                    
                    soup2 = BeautifulSoup(browser.page_source, "lxml")

                    # 메인룬, 서브룬
                    rune_stat_lst.append(soup2.find_all("span", attrs={"class":"RuneTree__styleName"})[0].get_text())
                    rune_stat_lst.append(soup2.find_all("span", attrs={"class":"RuneTree__styleName"})[1].get_text())

                    # 세부 룬 정보
                    for k in soup2.find_all("div", attrs={"class":"RuneTree__rune RuneIcon"}):
                        rune_stat_lst.append(k.find("img")["alt"])

                    for k in soup2.find_all("div", attrs={"class":"StatPerkIcon RuneTree__perk"}):
                        rune_stat_lst.append(k.find("img")["alt"])

                    rune_df = pd.concat([rune_df, pd.DataFrame(rune_stat_lst).T], axis=0)

            # 기본 정보, 룬 정보 결합
            set_data = pd.concat([data, rune_df], axis=1)

            # 최종 데이터
            final_data = pd.concat([final_data, set_data], axis=0)
            set_num += 1
        except:
            browser.find_elements_by_xpath("//*[contains(text(), '전체경기')]")[0].click()
            soup4 = BeautifulSoup(browser.page_source, "lxml")
            final_data["url"] = soup4.find("iframe", attrs={"class":"YoutubeVideo"})["src"]
            browser.quit()
            break
    
    return final_data
```

- 우선 지금까지의 과정을 함수로서 정의합니다.


```python
def data_export(season):
    # 모든 매치 URL
    url_lst = match_url(want_season = season)
    print("총 매치수:", len(url_lst))

    # 경기 정보 쌓기
    stack_data = pd.DataFrame()

    total_duration = 0
    for i, url in enumerate(url_lst):
        start_time = time.time()

        data = league_data2(url)
        stack_data = pd.concat([stack_data, data], axis=0)

        duration = round(time.time() - start_time, 0)
        total_duration += duration
        print(f"{i}번째 경기 수집 완료: {duration}초")

    print(f"총 수집 시간: {round(total_duration/60,0)}분")

    # 컬럼명 수정
    stack_data.columns = ["champion", "level", "spell1", "spell2", "summonerName", "K", "D", "A", "TotalDamage",
                           "vision_score", "minion_cs", "jungle_cs", "total_cs", 
                           "item1", "item2", "item3", "item4", "item5", "item6", "item7", 
                           "ban", "side", "result", "turret", "inhibitor", "baron", "dragon", "riftHerald",
                           "length", "date", "version", 
                           "main_rune", "sub_rune", "rune1", "rune2", "rune3", "rune4", "rune5", "rune6",
                           "stat1", "stat2", "stat3", "url"]
    
    return stack_data
```

- 앞선 과정으로 특정 리그, 연도, 토너먼트의 모든 상세 경기 url을 수집하였습니다.


- 그리고 하나의 상세 경기에 대해 결과 정보를 수집해보았습니다.


- 이제는 이를 통합하여 전체 정보를 가져오면 됩니다.


- 기타 컬럼명 등의 수정 작업도 함께 진행합니다.


```python
season = "LCK 2021 Summer"
season2 = season.replace(" ", "_").replace("-", "_")
exec(f"{season2} = data_export(season)")
```

    총 매치수: 95
    0번째 경기 수집 완료: 43.0초
    1번째 경기 수집 완료: 43.0초
    2번째 경기 수집 완료: 61.0초
    3번째 경기 수집 완료: 59.0초
    4번째 경기 수집 완료: 61.0초
    5번째 경기 수집 완료: 60.0초
    6번째 경기 수집 완료: 58.0초
    7번째 경기 수집 완료: 46.0초
    8번째 경기 수집 완료: 45.0초
    9번째 경기 수집 완료: 45.0초
    10번째 경기 수집 완료: 62.0초
    11번째 경기 수집 완료: 57.0초
    12번째 경기 수집 완료: 46.0초
    13번째 경기 수집 완료: 45.0초
    14번째 경기 수집 완료: 45.0초
    15번째 경기 수집 완료: 60.0초
    16번째 경기 수집 완료: 43.0초
    17번째 경기 수집 완료: 59.0초
    18번째 경기 수집 완료: 43.0초
    19번째 경기 수집 완료: 58.0초
    20번째 경기 수집 완료: 44.0초
    21번째 경기 수집 완료: 59.0초
    22번째 경기 수집 완료: 60.0초
    23번째 경기 수집 완료: 49.0초
    24번째 경기 수집 완료: 43.0초
    25번째 경기 수집 완료: 57.0초
    26번째 경기 수집 완료: 48.0초
    27번째 경기 수집 완료: 44.0초
    28번째 경기 수집 완료: 45.0초
    29번째 경기 수집 완료: 58.0초
    30번째 경기 수집 완료: 58.0초
    31번째 경기 수집 완료: 59.0초
    32번째 경기 수집 완료: 44.0초
    33번째 경기 수집 완료: 59.0초
    34번째 경기 수집 완료: 59.0초
    35번째 경기 수집 완료: 42.0초
    36번째 경기 수집 완료: 62.0초
    37번째 경기 수집 완료: 60.0초
    38번째 경기 수집 완료: 58.0초
    39번째 경기 수집 완료: 44.0초
    40번째 경기 수집 완료: 46.0초
    41번째 경기 수집 완료: 50.0초
    42번째 경기 수집 완료: 60.0초
    43번째 경기 수집 완료: 56.0초
    44번째 경기 수집 완료: 41.0초
    45번째 경기 수집 완료: 60.0초
    46번째 경기 수집 완료: 60.0초
    47번째 경기 수집 완료: 44.0초
    48번째 경기 수집 완료: 44.0초
    49번째 경기 수집 완료: 43.0초
    50번째 경기 수집 완료: 42.0초
    51번째 경기 수집 완료: 61.0초
    52번째 경기 수집 완료: 45.0초
    53번째 경기 수집 완료: 44.0초
    54번째 경기 수집 완료: 58.0초
    55번째 경기 수집 완료: 60.0초
    56번째 경기 수집 완료: 60.0초
    57번째 경기 수집 완료: 44.0초
    58번째 경기 수집 완료: 60.0초
    59번째 경기 수집 완료: 60.0초
    60번째 경기 수집 완료: 44.0초
    61번째 경기 수집 완료: 62.0초
    62번째 경기 수집 완료: 41.0초
    63번째 경기 수집 완료: 60.0초
    64번째 경기 수집 완료: 59.0초
    65번째 경기 수집 완료: 61.0초
    66번째 경기 수집 완료: 61.0초
    67번째 경기 수집 완료: 43.0초
    68번째 경기 수집 완료: 60.0초
    69번째 경기 수집 완료: 46.0초
    70번째 경기 수집 완료: 59.0초
    71번째 경기 수집 완료: 61.0초
    72번째 경기 수집 완료: 60.0초
    73번째 경기 수집 완료: 44.0초
    74번째 경기 수집 완료: 44.0초
    75번째 경기 수집 완료: 44.0초
    76번째 경기 수집 완료: 61.0초
    77번째 경기 수집 완료: 60.0초
    78번째 경기 수집 완료: 42.0초
    79번째 경기 수집 완료: 57.0초
    80번째 경기 수집 완료: 41.0초
    81번째 경기 수집 완료: 46.0초
    82번째 경기 수집 완료: 43.0초
    83번째 경기 수집 완료: 60.0초
    84번째 경기 수집 완료: 59.0초
    85번째 경기 수집 완료: 45.0초
    86번째 경기 수집 완료: 44.0초
    87번째 경기 수집 완료: 61.0초
    88번째 경기 수집 완료: 44.0초
    89번째 경기 수집 완료: 44.0초
    90번째 경기 수집 완료: 75.0초
    91번째 경기 수집 완료: 57.0초
    92번째 경기 수집 완료: 61.0초
    93번째 경기 수집 완료: 77.0초
    94번째 경기 수집 완료: 76.0초
    총 수집 시간: 84.0분
    

- 보시다시피 시간이 매우 오래 걸립니다.


- 아무래도 `selnium`으로 정보를 확인하는 점에서 감안해야할 부분입니다.


- 다만 현재는 특정 시즌 전체 정보를 수집하기에 하나의 매치 단위로는 대략 1분 정도면 괜찮아 보입니다.


- 필요에 따라 매치 단위 url을 넣는 방식으로 사용하면 쉽게 정보를 확인할 수 있겠네요.


- 혹은 룬 정보 등을 포기한다거나 할 수도 있겠어요.


```python
LCK_2021_Summer
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>champion</th>
      <th>level</th>
      <th>spell1</th>
      <th>spell2</th>
      <th>summonerName</th>
      <th>K</th>
      <th>D</th>
      <th>A</th>
      <th>TotalDamage</th>
      <th>vision_score</th>
      <th>...</th>
      <th>rune1</th>
      <th>rune2</th>
      <th>rune3</th>
      <th>rune4</th>
      <th>rune5</th>
      <th>rune6</th>
      <th>stat1</th>
      <th>stat2</th>
      <th>stat3</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>나르</td>
      <td>18</td>
      <td>점멸</td>
      <td>순간이동</td>
      <td>BRO Hoya</td>
      <td>2</td>
      <td>0</td>
      <td>9</td>
      <td>19967</td>
      <td>46</td>
      <td>...</td>
      <td>착취의 손아귀</td>
      <td>철거</td>
      <td>뼈 방패</td>
      <td>과잉성장</td>
      <td>피의 맛</td>
      <td>굶주린 사냥꾼</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>릴리아</td>
      <td>16</td>
      <td>강타</td>
      <td>점멸</td>
      <td>BRO UmTi</td>
      <td>2</td>
      <td>3</td>
      <td>13</td>
      <td>11555</td>
      <td>55</td>
      <td>...</td>
      <td>난입</td>
      <td>빛의 망토</td>
      <td>기민함</td>
      <td>물 위를 걷는 자</td>
      <td>외상</td>
      <td>우주적 통찰력</td>
      <td>5008</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>루시안</td>
      <td>18</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>BRO Lava</td>
      <td>11</td>
      <td>0</td>
      <td>5</td>
      <td>22265</td>
      <td>71</td>
      <td>...</td>
      <td>집중 공격</td>
      <td>침착</td>
      <td>전설: 핏빛 길</td>
      <td>최후의 저항</td>
      <td>비스킷 배달</td>
      <td>시간 왜곡 물약</td>
      <td>5005</td>
      <td>5008</td>
      <td>5003</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>칼리스타</td>
      <td>17</td>
      <td>회복</td>
      <td>점멸</td>
      <td>BRO Hena</td>
      <td>3</td>
      <td>1</td>
      <td>10</td>
      <td>9735</td>
      <td>87</td>
      <td>...</td>
      <td>칼날비</td>
      <td>피의 맛</td>
      <td>좀비 와드</td>
      <td>굶주린 사냥꾼</td>
      <td>비스킷 배달</td>
      <td>우주적 통찰력</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>0</th>
      <td>그라가스</td>
      <td>14</td>
      <td>점화</td>
      <td>점멸</td>
      <td>BRO Delight</td>
      <td>1</td>
      <td>2</td>
      <td>11</td>
      <td>5349</td>
      <td>111</td>
      <td>...</td>
      <td>여진</td>
      <td>생명의 샘</td>
      <td>재생의 바람</td>
      <td>불굴의 의지</td>
      <td>비스킷 배달</td>
      <td>우주적 통찰력</td>
      <td>5007</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/rFE21sDpHjs</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>0</th>
      <td>제이스</td>
      <td>18</td>
      <td>점멸</td>
      <td>순간이동</td>
      <td>DK Khan</td>
      <td>5</td>
      <td>3</td>
      <td>4</td>
      <td>24782</td>
      <td>56</td>
      <td>...</td>
      <td>난입</td>
      <td>마나순환 팔찌</td>
      <td>절대 집중</td>
      <td>폭풍의 결집</td>
      <td>마법의 신발</td>
      <td>비스킷 배달</td>
      <td>5008</td>
      <td>5008</td>
      <td>5003</td>
      <td>https://www.youtube.com/embed/m5IY7xwPJW8</td>
    </tr>
    <tr>
      <th>0</th>
      <td>트런들</td>
      <td>14</td>
      <td>강타</td>
      <td>점멸</td>
      <td>DK Canyon</td>
      <td>0</td>
      <td>2</td>
      <td>6</td>
      <td>4040</td>
      <td>50</td>
      <td>...</td>
      <td>집중 공격</td>
      <td>승전보</td>
      <td>전설: 강인함</td>
      <td>최후의 일격</td>
      <td>마법의 신발</td>
      <td>우주적 통찰력</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/m5IY7xwPJW8</td>
    </tr>
    <tr>
      <th>0</th>
      <td>르블랑</td>
      <td>17</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>DK ShowMaker</td>
      <td>7</td>
      <td>1</td>
      <td>6</td>
      <td>26242</td>
      <td>95</td>
      <td>...</td>
      <td>감전</td>
      <td>피의 맛</td>
      <td>좀비 와드</td>
      <td>굶주린 사냥꾼</td>
      <td>마나순환 팔찌</td>
      <td>깨달음</td>
      <td>5005</td>
      <td>5008</td>
      <td>5003</td>
      <td>https://www.youtube.com/embed/m5IY7xwPJW8</td>
    </tr>
    <tr>
      <th>0</th>
      <td>애쉬</td>
      <td>16</td>
      <td>정화</td>
      <td>점멸</td>
      <td>DK Ghost</td>
      <td>2</td>
      <td>0</td>
      <td>9</td>
      <td>16316</td>
      <td>82</td>
      <td>...</td>
      <td>칼날비</td>
      <td>피의 맛</td>
      <td>좀비 와드</td>
      <td>굶주린 사냥꾼</td>
      <td>비스킷 배달</td>
      <td>쾌속 접근</td>
      <td>5005</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/m5IY7xwPJW8</td>
    </tr>
    <tr>
      <th>0</th>
      <td>세라핀</td>
      <td>12</td>
      <td>탈진</td>
      <td>점멸</td>
      <td>DK BeryL</td>
      <td>0</td>
      <td>3</td>
      <td>4</td>
      <td>7069</td>
      <td>103</td>
      <td>...</td>
      <td>수호자</td>
      <td>보호막 강타</td>
      <td>뼈 방패</td>
      <td>소생</td>
      <td>비스킷 배달</td>
      <td>우주적 통찰력</td>
      <td>5008</td>
      <td>5008</td>
      <td>5002</td>
      <td>https://www.youtube.com/embed/m5IY7xwPJW8</td>
    </tr>
  </tbody>
</table>
<p>2440 rows × 43 columns</p>
</div>


