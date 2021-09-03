---
title: "[OPGG] 페이커 게임 정보 스크래핑 (3)"
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

# 1. 페이커 신드라 정보

이번엔 모든 챔피언이 아닌 신드라에 대한 정보만 스크래핑 할 것이다.

큰 차이는 없으며 selenium으로 신드라를 입력하고 클릭하는 명령을 추가해주었다.


```python
import requests
import time
import pandas as pd
from bs4 import BeautifulSoup
from selenium import webdriver
```


```python
# selenium을 백그라운드로 실행하기 위해 옵션 설정
options = webdriver.ChromeOptions()
options.add_argument('headless')
options.add_argument('window-size=1920x1080')
```


```python
# 우리가 데이터를 가져올 웹 페이지입니다.
faker_opgg_url = 'https://www.op.gg/summoner/userName=Hide%20on%20bush'

# selenium이 제어할 chrome을 실행합니다.
# driver = webdriver.Chrome("./chromedriver")
driver = webdriver.Chrome("./chromedriver", options= options)
```


```python
# 데이터를 가져올 페이지로 이동합니다.
driver.get(faker_opgg_url)
```


```python
# 챔피언 검색창에 'Syndra' 입력
driver.find_element_by_id('right_search_champion').find_element_by_css_selector('input.Input').send_keys('신드라\n')
time.sleep(1)
```


```python
# 매치 탭에서 'Ranked Solo' 클릭
driver.find_element_by_id('right_gametype_soloranked').click()
time.sleep(1)
```


```python
# '더 보기' 버튼이 활성화되어있으면(리스트의 끝까지 도달하지 않았으면) 계속 클릭하여 이전 게임을 모두 로딩합니다.
while True:
    try:
        driver.find_element_by_css_selector('.GameMoreButton').click()
        # 게임 로딩, html 코드 변경까지 여유 시간을 1초 가집니다.
        time.sleep(1)
        
    # 에러가 나면(페이지에서 '더 보기' 버튼이 없을 경우) while문을 탈출합니다.
    except Exception as e:
        pass
        break
```


```python
# 결과가 들어갈 빈 리스트를 만듭니다.
faker_champions = []
faker_kills = []
faker_deaths = []
faker_assists = []
faker_results = []
```


```python
# 각 게임에 대해 웹 페이지에 기재된 스탯을 찾아서(selector 사용) 결과 리스트에 append하기
for gamelist in driver.find_elements_by_css_selector('div.GameItemList'):
    for game in gamelist.find_elements_by_css_selector('div.GameItemWrap'):
        faker_champions.append(game.find_element_by_css_selector('div.ChampionName').text.strip())
        faker_kills.append(game.find_element_by_css_selector('div.KDA').
                           find_element_by_css_selector('div.KDA').
                           find_element_by_css_selector('span.Kill').text.strip())
        faker_deaths.append(game.find_element_by_css_selector('div.KDA').
                           find_element_by_css_selector('div.KDA').
                           find_element_by_css_selector('span.Death').text.strip())
        faker_assists.append(game.find_element_by_css_selector('div.KDA').
                           find_element_by_css_selector('div.KDA').
                           find_element_by_css_selector('span.Assist').text.strip())
        faker_results.append(game.find_element_by_css_selector('div.GameStats').
                           find_element_by_css_selector('div.GameResult').text.strip())
```

- `page_source`를 사용해서 `requests` 패키지를 사용하는 대신 셀레니움 자체에서 진행


- gamelist에는 한 세트(20개의 대전 정보)가 있고 이중 for문 형태


```python
# selenium이 제어하는 크롬을 종료합니다.
driver.quit()
```


```python
# DataFrame으로 변환 후 출력
faker_total_df = pd.DataFrame([faker_champions,
                                faker_results,
                                faker_kills,
                                faker_deaths,
                                faker_assists],
                               index = ['champion', 'result', 'kills', 'deaths', 'assists']).T
faker_total_df
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
      <th>result</th>
      <th>kills</th>
      <th>deaths</th>
      <th>assists</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>신드라</td>
      <td>승리</td>
      <td>5</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>신드라</td>
      <td>승리</td>
      <td>7</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>신드라</td>
      <td>승리</td>
      <td>5</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>신드라</td>
      <td>패배</td>
      <td>6</td>
      <td>4</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>신드라</td>
      <td>승리</td>
      <td>5</td>
      <td>4</td>
      <td>9</td>
    </tr>
    <tr>
      <th>5</th>
      <td>신드라</td>
      <td>승리</td>
      <td>8</td>
      <td>3</td>
      <td>17</td>
    </tr>
    <tr>
      <th>6</th>
      <td>신드라</td>
      <td>패배</td>
      <td>5</td>
      <td>10</td>
      <td>10</td>
    </tr>
    <tr>
      <th>7</th>
      <td>신드라</td>
      <td>패배</td>
      <td>4</td>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>8</th>
      <td>신드라</td>
      <td>승리</td>
      <td>7</td>
      <td>4</td>
      <td>10</td>
    </tr>
    <tr>
      <th>9</th>
      <td>신드라</td>
      <td>승리</td>
      <td>6</td>
      <td>6</td>
      <td>9</td>
    </tr>
    <tr>
      <th>10</th>
      <td>신드라</td>
      <td>승리</td>
      <td>17</td>
      <td>4</td>
      <td>5</td>
    </tr>
    <tr>
      <th>11</th>
      <td>신드라</td>
      <td>패배</td>
      <td>3</td>
      <td>10</td>
      <td>2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>신드라</td>
      <td>승리</td>
      <td>6</td>
      <td>6</td>
      <td>10</td>
    </tr>
    <tr>
      <th>13</th>
      <td>신드라</td>
      <td>패배</td>
      <td>5</td>
      <td>4</td>
      <td>2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>신드라</td>
      <td>패배</td>
      <td>0</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <th>15</th>
      <td>신드라</td>
      <td>패배</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>신드라</td>
      <td>승리</td>
      <td>3</td>
      <td>3</td>
      <td>6</td>
    </tr>
    <tr>
      <th>17</th>
      <td>신드라</td>
      <td>승리</td>
      <td>15</td>
      <td>6</td>
      <td>13</td>
    </tr>
    <tr>
      <th>18</th>
      <td>신드라</td>
      <td>패배</td>
      <td>5</td>
      <td>4</td>
      <td>10</td>
    </tr>
    <tr>
      <th>19</th>
      <td>신드라</td>
      <td>승리</td>
      <td>4</td>
      <td>4</td>
      <td>6</td>
    </tr>
    <tr>
      <th>20</th>
      <td>신드라</td>
      <td>승리</td>
      <td>8</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>21</th>
      <td>신드라</td>
      <td>패배</td>
      <td>7</td>
      <td>7</td>
      <td>4</td>
    </tr>
    <tr>
      <th>22</th>
      <td>신드라</td>
      <td>승리</td>
      <td>1</td>
      <td>5</td>
      <td>8</td>
    </tr>
    <tr>
      <th>23</th>
      <td>신드라</td>
      <td>패배</td>
      <td>4</td>
      <td>9</td>
      <td>5</td>
    </tr>
    <tr>
      <th>24</th>
      <td>신드라</td>
      <td>승리</td>
      <td>3</td>
      <td>2</td>
      <td>9</td>
    </tr>
    <tr>
      <th>25</th>
      <td>신드라</td>
      <td>패배</td>
      <td>5</td>
      <td>7</td>
      <td>3</td>
    </tr>
    <tr>
      <th>26</th>
      <td>신드라</td>
      <td>승리</td>
      <td>4</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>27</th>
      <td>신드라</td>
      <td>승리</td>
      <td>6</td>
      <td>2</td>
      <td>8</td>
    </tr>
    <tr>
      <th>28</th>
      <td>신드라</td>
      <td>패배</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>29</th>
      <td>신드라</td>
      <td>패배</td>
      <td>4</td>
      <td>6</td>
      <td>9</td>
    </tr>
    <tr>
      <th>30</th>
      <td>신드라</td>
      <td>승리</td>
      <td>7</td>
      <td>2</td>
      <td>14</td>
    </tr>
    <tr>
      <th>31</th>
      <td>신드라</td>
      <td>패배</td>
      <td>4</td>
      <td>5</td>
      <td>6</td>
    </tr>
    <tr>
      <th>32</th>
      <td>신드라</td>
      <td>패배</td>
      <td>2</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>33</th>
      <td>신드라</td>
      <td>승리</td>
      <td>3</td>
      <td>4</td>
      <td>6</td>
    </tr>
    <tr>
      <th>34</th>
      <td>신드라</td>
      <td>패배</td>
      <td>2</td>
      <td>5</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>


