---
title: "[OPGG] 페이커 게임 정보 스크래핑 (2)"
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

# 1. Selenium

selenium은 직접 웹 사이트를 클릭하고 검색하는 등의 기능이 가능하다.

앞서 페이커의 최근 20게임을 가져왔는데 이번엔 전체 정보를 가져와보자.


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

- headless 옵션을 사용해서 웹 실행 없이 동작을 수행한다.


```python
# 데이터를 가져올 페이지로 이동합니다.
driver.get(faker_opgg_url)
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

- 최근 20게임만 뜨는 사이트에서 "더 보기" 버튼을 눌러 모든 게임 정보를 확인한다.


- 실제 더보기 버튼의 class는 "GameMoreButton Box"라 적혀있다.


- "GameMoreButton Box"에서 공백은 and를 의미하므로 두 개의 클래스가 같이 있는 것이다. 


- 그렇기에 '.GameMoreButton'으로 기입하였다.


```python
# 게임이 모두 로딩된 페이지의 html 코드를 faker_total_html로 저장합니다.
faker_total_html = driver.page_source

# selenium이 제어하는 크롬을 종료합니다.
driver.quit()
```

- 모두 로딩된 페이지의 HTML 코드를 가져온 후 크롬 드라이버를 종료하였다.


- 이후 과정은 페이커의 최근 20게임을 가져온 것과 동일하다.


```python
# html 형식에 맞춰 파싱(parsing; 추후 이용하기 쉽도록 쪼개기)합니다.
faker_total_soup = BeautifulSoup(faker_total_html, 'html.parser')
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
# 전체 html 코드 중 우리가 원하는 selector를 만족하는 것만 가져오기
faker_total_games_html = faker_total_soup.select('div.GameItemList div.GameItemWrap')

# 그렇게 가져온 html 코드의 element 개수 == 끝까지 로딩된 모든 게임 수 출력
total_game_len = len(faker_total_games_html)
print(total_game_len)
```

    691
    


```python
# 각 게임에 대해 웹 페이지에 기재된 스탯을 찾아서(selector 사용) 결과 리스트에 append하기
for i in range(total_game_len):
    faker_champions.append(list(faker_total_games_html[i].select_one('div.ChampionName').
                                               stripped_strings)[0])
    faker_kills.append(list(faker_total_games_html[i].select_one('div.KDA div.KDA span.Kill').
                                               stripped_strings)[0])
    faker_deaths.append(list(faker_total_games_html[i].select_one('div.KDA div.KDA span.Death').
                                               stripped_strings)[0])
    faker_assists.append(list(faker_total_games_html[i].select_one('div.KDA div.KDA span.Assist').
                                               stripped_strings)[0])
    faker_results.append(list(faker_total_games_html[i].select_one('div.GameStats div.GameResult').
                                               stripped_strings)[0])
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
      <td>오리아나</td>
      <td>승리</td>
      <td>10</td>
      <td>3</td>
      <td>18</td>
    </tr>
    <tr>
      <th>1</th>
      <td>리 신</td>
      <td>승리</td>
      <td>8</td>
      <td>4</td>
      <td>14</td>
    </tr>
    <tr>
      <th>2</th>
      <td>애니비아</td>
      <td>패배</td>
      <td>7</td>
      <td>5</td>
      <td>10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>르블랑</td>
      <td>승리</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>블리츠크랭크</td>
      <td>승리</td>
      <td>4</td>
      <td>1</td>
      <td>17</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>686</th>
      <td>비에고</td>
      <td>패배</td>
      <td>4</td>
      <td>6</td>
      <td>5</td>
    </tr>
    <tr>
      <th>687</th>
      <td>비에고</td>
      <td>승리</td>
      <td>6</td>
      <td>1</td>
      <td>12</td>
    </tr>
    <tr>
      <th>688</th>
      <td>사일러스</td>
      <td>승리</td>
      <td>10</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>689</th>
      <td>그웬</td>
      <td>승리</td>
      <td>6</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>690</th>
      <td>그웬</td>
      <td>승리</td>
      <td>6</td>
      <td>5</td>
      <td>10</td>
    </tr>
  </tbody>
</table>
<p>691 rows × 5 columns</p>
</div>


