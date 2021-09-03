---
title: "[OPGG] 특정 유저, 시즌별 챔피언 정보"
excerpt: "데이터 분석가 양성 과정 3주차"
categories: 
  - OPGG
tags: 
    - Python
    - OPGG
toc: TRUE
toc_sticky: TRUE
use_math: TRUE
---

# 1. 시즌 챔피언 정보

앞서 배웠던 웹 스크래핑 내용을 토대로 특정 유저의 특정 시즌 랭크 플레이 챔피언 정보를 스크래핑 해보았다.


```python
import time
import pandas as pd

import requests
from bs4 import BeautifulSoup
from selenium import webdriver
```


```python
# headless selenium
options = webdriver.ChromeOptions()
options.add_argument('headless')
options.add_argument('window-size=1920x1080')
```


```python
def my_champion(userName, season):
    # 빈 데이터 프레임
    data = pd.DataFrame()
    
    # 띄어쓰기 있는 유저이면 +로 구분
    userName2 = "+".join(userName.split())

    # 유저별 검색 결과 url
    url = f"https://www.op.gg/summoner/userName={userName2}"

    # driver 실행
    # driver = webdriver.Chrome("./chromedriver")
    driver = webdriver.Chrome("./chromedriver", options= options)
    driver.get(url)
    time.sleep(1)

    # 챔피언 클릭 후 다음 창 로드까지 잠시 대기
    driver.find_element_by_id("left_champion").find_element_by_css_selector("a").click()
    time.sleep(1)

    # 원하는 시즌 설정 (xpath의 li[index]를 사용: index가 1부터 시작)
    # 시즌이 추가될 때마다 수정해주어야 함
    season_dict = {
        2021:"tabItem season-17",
        2020:"tabItem season-15",
        2019:"tabItem season-13",
        2018:"tabItem season-11",
        2017:"tabItem season-7",
        2016:"tabItem season-6",
        2015:"tabItem season-5",
        2014:"tabItem season-4",
        2013:"tabItem season-3",
        2012:"tabItem season-2",
        2011:"tabItem season-1"
    }

    season2 = list(season_dict.keys()).index(season) + 1

    # 시즌 클릭 (모두 클릭해야 정보가 나타나므로)
    for i in range(len(season_dict)):
        driver.find_element_by_xpath(f"//*[@id='champion_season']/li[{i+1}]/a").click()

    # url 정보 저장
    soup = BeautifulSoup(driver.page_source, "lxml")
    driver.quit()

    # 플레이한 챔피언 데이터 테이블
    champ_info = soup.find("div", attrs= {"class":f"{season_dict[season]}"})
    champ_info2 = champ_info.find("tbody", attrs= {"class":"Body"})
    champ_info2 = champ_info2.find_all("tr")
    
    # 각 태그 지정해서 원하는 정보 추출
    for i in range(len(champ_info2)):
        lst = []
        # 챔피언 이름
        lst.append(champ_info2[i].find("td", attrs= {"class":"ChampionName Cell"}).get_text().strip())

        # 승/패 (단, 한번도 이기거나 진적이 없으면 해당 class가 없음)
        if champ_info2[i].find("div", attrs= {"class":"Text Left"}) == None:
            lst.append(0)
        else:
            lst.append(champ_info2[i].find("div", attrs= {"class":"Text Left"}).get_text().strip()[:-1])

        if champ_info2[i].find("div", attrs= {"class":"Text Right"}) == None:
            lst.append(0)
        else:
            lst.append(champ_info2[i].find("div", attrs= {"class":"Text Right"}).get_text().strip()[:-1])

        # K/D/A
        KDA = champ_info2[i].find("div", attrs= {"class":"KDA"}).get_text().strip().split()
        lst.append(KDA[0])
        lst.append(KDA[2])
        lst.append(KDA[4])
        
        # 기타 정보
        else_info = champ_info2[i].find_all("td", attrs={"class":"Value Cell"})
        for i in else_info:
            info = i.get_text().strip().split()
            if len(info) == 0:
                lst.append(0)
            else:
                lst.append(info[0])
        
        # 데이터 프레임
        champ_df = pd.DataFrame(lst)
        data = pd.concat([data, champ_df], axis=1)
    
    # 데이터 프레임 형태, 타입 설정
    data = data.T
    data.index = range(data.shape[0])
    data.columns = [
        "champion", "win", "lose", "kill", "death", "assist", 
        "gold", "cs", "max_kill", "max_death", "avg_damage", "avg_damaged", "double", "triple", "quadra", "penta"
    ]
    
    data["win"] = data["win"].astype(int)
    data["lose"] = data["lose"].astype(int)
    data["kill"] = data["kill"].astype(float)
    data["death"] = data["death"].astype(float)
    data["assist"] = data["assist"].astype(float)
    data["gold"] = data["gold"].apply(lambda x: int(x.replace(",","")))
    data["cs"] = data["cs"].astype(float)
    data["max_kill"] = data["max_kill"].astype(int)
    data["max_death"] = data["max_death"].astype(int)
    data["avg_damage"] = data["avg_damage"].apply(lambda x: int(x.replace(",","")))
    data["avg_damaged"] = data["avg_damaged"].apply(lambda x: int(x.replace(",","")))
    data["double"] = data["double"].astype(int)
    data["triple"] = data["triple"].astype(int)
    data["quadra"] = data["quadra"].astype(int)
    data["penta"] = data["penta"].astype(int)
    
    return data
```

- 유저 이름에 띄어쓰기가 있으면 검색시 URL이 `userName=Hide+on+bush`과 같이 +로 구분되어 이를 반영


- 챔피언 탭 클릭 후 각 시즌을 모두 클릭해야만 해당 정보가 생성되므로 모두 클릭(노말 제외)


- 시즌 입력시 연도로 입력하고 이를 URL 반영하기 위해 dictionary를 생성


- 만약 특정 챔피언이 모두 승이거나 모두 패이면 해당 class를 가지는 div 태그가 없음


- 더블킬, 트리플킬 등의 경우는 없으면 해당 class의 태그는 존재하지만 텍스트가 빈 값으로 입력되있음

OP.GG: **현재 리그오브레전드 공식 API 서버의 문제로 인해 최근 게임 데이터 갱신이 조금 늦을 수 있습니다.**

위 메시지 이후로 챔피언 탭 클릭 명령어 

```python
driver.find_element_by_id("left_champion").find_element_by_css_selector("a").click()
```

가 될 때도 있고 안 될때도 있음 (확실한 이유는..?)


```python
# 원하는 유저의 특정 시즌 챔피언 정보
userName = "Hide on bush"
season = 2020

result = my_champion(userName, season)
result
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
      <th>win</th>
      <th>lose</th>
      <th>kill</th>
      <th>death</th>
      <th>assist</th>
      <th>gold</th>
      <th>cs</th>
      <th>max_kill</th>
      <th>max_death</th>
      <th>avg_damage</th>
      <th>avg_damaged</th>
      <th>double</th>
      <th>triple</th>
      <th>quadra</th>
      <th>penta</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>사일러스</td>
      <td>55</td>
      <td>60</td>
      <td>6.0</td>
      <td>4.3</td>
      <td>5.6</td>
      <td>11307</td>
      <td>202.5</td>
      <td>16</td>
      <td>13</td>
      <td>124393</td>
      <td>25284</td>
      <td>78</td>
      <td>14</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>조이</td>
      <td>54</td>
      <td>50</td>
      <td>5.7</td>
      <td>3.3</td>
      <td>6.8</td>
      <td>11623</td>
      <td>210.0</td>
      <td>19</td>
      <td>9</td>
      <td>136912</td>
      <td>13779</td>
      <td>53</td>
      <td>6</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>루시안</td>
      <td>53</td>
      <td>51</td>
      <td>5.0</td>
      <td>4.1</td>
      <td>5.3</td>
      <td>12133</td>
      <td>234.6</td>
      <td>15</td>
      <td>11</td>
      <td>156282</td>
      <td>16874</td>
      <td>44</td>
      <td>5</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>아칼리</td>
      <td>37</td>
      <td>47</td>
      <td>6.5</td>
      <td>4.0</td>
      <td>4.2</td>
      <td>11326</td>
      <td>209.8</td>
      <td>24</td>
      <td>9</td>
      <td>121981</td>
      <td>21255</td>
      <td>65</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>세트</td>
      <td>47</td>
      <td>26</td>
      <td>4.5</td>
      <td>3.0</td>
      <td>6.5</td>
      <td>10649</td>
      <td>187.1</td>
      <td>13</td>
      <td>7</td>
      <td>112374</td>
      <td>24356</td>
      <td>30</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
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
    </tr>
    <tr>
      <th>95</th>
      <td>신 짜오</td>
      <td>0</td>
      <td>1</td>
      <td>7.0</td>
      <td>8.0</td>
      <td>15.0</td>
      <td>13612</td>
      <td>177.0</td>
      <td>7</td>
      <td>8</td>
      <td>181530</td>
      <td>52539</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>96</th>
      <td>룰루</td>
      <td>0</td>
      <td>1</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>20.0</td>
      <td>11502</td>
      <td>183.0</td>
      <td>2</td>
      <td>6</td>
      <td>107024</td>
      <td>21541</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>97</th>
      <td>퀸</td>
      <td>0</td>
      <td>1</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>5193</td>
      <td>119.0</td>
      <td>0</td>
      <td>3</td>
      <td>37514</td>
      <td>8179</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>98</th>
      <td>스웨인</td>
      <td>0</td>
      <td>1</td>
      <td>8.0</td>
      <td>5.0</td>
      <td>7.0</td>
      <td>14101</td>
      <td>213.0</td>
      <td>8</td>
      <td>5</td>
      <td>140346</td>
      <td>36175</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>99</th>
      <td>제라스</td>
      <td>0</td>
      <td>1</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>10.0</td>
      <td>9509</td>
      <td>33.0</td>
      <td>4</td>
      <td>6</td>
      <td>62670</td>
      <td>16061</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 16 columns</p>
</div>


