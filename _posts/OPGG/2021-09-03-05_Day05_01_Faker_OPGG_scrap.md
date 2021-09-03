---
title: "[OPGG] 페이커 게임 정보 웹 스크래핑 (1)"
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

# 1. 웹 스크래핑

이번 강의에서는 웹 스크래핑에 대해 배웠다.

여기선 페이커의 플레이 정보를 OPGG 웹 사이트에서 불러올 것이다.


```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

# 우리가 데이터를 가져올 웹 페이지입니다.
faker_opgg_url = 'https://www.op.gg/summoner/userName=Hide%20on%20bush'

# 페이지의 html 코드를 faker_html로 저장합니다.
faker_html = requests.get(faker_opgg_url).text

# html 형식에 맞춰 파싱(parsing; 추후 이용하기 쉽도록 쪼개기)합니다.
faker_soup = BeautifulSoup(faker_html, "html.parser")
```

- `BeautifulSoup`을 이용한 웹 스크래핑은 이전에 [나도코딩 웹 스크래핑](https://romg2.github.io/categories/%EB%82%98%EB%8F%84%EC%BD%94%EB%94%A9)에서 공부하여 정리한 경험이 있다.


```python
# faker_soup
```

- HTML 파일을 확인 가능하나 매우 기니까 원한다면 주석 해제하여 구조를 살펴보자.


```python
# 결과가 들어갈 빈 리스트를 만듭니다.
faker_recent_champions = []
faker_recent_kills = []
faker_recent_deaths = []
faker_recent_assists = []
faker_recent_results = []
```


```python
# 전체 html 코드 중 우리가 원하는 selector를 만족하는 것만 가져오기
faker_recent_games_html = faker_soup.select('div.GameItemList div.GameItemWrap')

# 그렇게 가져온 html 코드의 element 개수 == 한 페이지에 최초로 로딩된 최근 게임 수 출력
recent_game_len = len(faker_recent_games_html)
print(recent_game_len)
```

    20
    

- OPGG에선 한 페이지에 최근 20게임이 로드된다.


- `select()`등 워낙 다양한 메소드가 많으니 직접 해보면서 찾아보는 것이 중요하다.


```python
type(faker_recent_games_html)
```




    list



- `select()` 사용하면 리스트 형태로 저장된다.


```python
# 각 게임에 대해 웹 페이지에 기재된 스탯을 찾아서(selector 사용) 결과 리스트에 append하기
for i in range(recent_game_len):
    faker_recent_champions.append(''.join(list(faker_recent_games_html[i].select('div.ChampionName')[0].
                                               stripped_strings)))
    faker_recent_kills.append(list(faker_recent_games_html[i].select('div.KDA div.KDA span.Kill')[0].
                                               stripped_strings)[0])
    faker_recent_deaths.append(list(faker_recent_games_html[i].select_one('div.KDA div.KDA span.Death').
                                               stripped_strings)[0])
    faker_recent_assists.append(list(faker_recent_games_html[i].select('div.KDA div.KDA span.Assist')[0].
                                               stripped_strings)[0])
    faker_recent_results.append(list(faker_recent_games_html[i].select('div.GameStats div.GameResult')[0].
                                               stripped_strings)[0])
#     아래 코드는 data-game-result라는 attr 가져오는 방법 다만 다시하기도 lose라 되있어서 사용 x
#     faker_recent_results.append(faker_recent_games_html[i].select_one('div.GameItem')['data-game-result'])
```

- 각 태그들은 본인이 직접 F12(개발자 모드)로 확인하면서 지정하면 된다.


```python
# DataFrame으로 변환 후 출력
faker_recent_df = pd.DataFrame([faker_recent_champions,
                                faker_recent_results,
                                faker_recent_kills,
                                faker_recent_deaths,
                                faker_recent_assists],
                               index = ['champion', 'result', 'kills', 'deaths', 'assists']).T
faker_recent_df
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
      <td>Orianna</td>
      <td>Victory</td>
      <td>10</td>
      <td>3</td>
      <td>18</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Lee Sin</td>
      <td>Victory</td>
      <td>8</td>
      <td>4</td>
      <td>14</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anivia</td>
      <td>Defeat</td>
      <td>7</td>
      <td>5</td>
      <td>10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>LeBlanc</td>
      <td>Victory</td>
      <td>4</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Blitzcrank</td>
      <td>Victory</td>
      <td>4</td>
      <td>1</td>
      <td>17</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Orianna</td>
      <td>Defeat</td>
      <td>3</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Orianna</td>
      <td>Defeat</td>
      <td>2</td>
      <td>7</td>
      <td>3</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Tristana</td>
      <td>Victory</td>
      <td>9</td>
      <td>8</td>
      <td>9</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Vayne</td>
      <td>Victory</td>
      <td>8</td>
      <td>5</td>
      <td>10</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Aphelios</td>
      <td>Victory</td>
      <td>5</td>
      <td>7</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Orianna</td>
      <td>Victory</td>
      <td>6</td>
      <td>5</td>
      <td>9</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Xerath</td>
      <td>Victory</td>
      <td>9</td>
      <td>2</td>
      <td>9</td>
    </tr>
    <tr>
      <th>12</th>
      <td>LeBlanc</td>
      <td>Victory</td>
      <td>9</td>
      <td>2</td>
      <td>12</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Zed</td>
      <td>Victory</td>
      <td>4</td>
      <td>5</td>
      <td>7</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Ryze</td>
      <td>Defeat</td>
      <td>4</td>
      <td>8</td>
      <td>4</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Irelia</td>
      <td>Defeat</td>
      <td>2</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Varus</td>
      <td>Victory</td>
      <td>5</td>
      <td>2</td>
      <td>6</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Renekton</td>
      <td>Victory</td>
      <td>11</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Orianna</td>
      <td>Remake</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Orianna</td>
      <td>Victory</td>
      <td>10</td>
      <td>5</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>



- 페이커의 최근 플레이한 정보를 위와 같이 OPGG 사이트에서 잘 불러왔다.


- 아래 코드들은 출력 형태에 대해 파악해보려고 작성한 것이다.


```python
faker_recent_games_html[0].select('div.ChampionName')
```




    [<div class="ChampionName">
     <a href="/champion/orianna/statistics" target="_blank">Orianna</a>
     </div>]




```python
faker_recent_games_html[0].find("div", attrs={"class":"ChampionName"})
```




    <div class="ChampionName">
    <a href="/champion/orianna/statistics" target="_blank">Orianna</a>
    </div>



- `find()` 메소드를 사용해도 된다.


- `select()`와는 다르게 리스트 형태가 아닌 기존 값?을 그대로 출력한다.


```python
faker_recent_games_html[0].select('div.ChampionName')[0].get_text()
```




    '\nOrianna\n'



- `select()`의 리스트에서 첫 번째 값을 `get_text()`하면 위와 같은 형태이다.


```python
list(faker_recent_games_html[0].select('div.ChampionName')[0].strings)
```




    ['\n', 'Orianna', '\n']



- `strings`를 사용하면 다음과 같이 분리되어 리스트로 나타난다.


```python
list(faker_recent_games_html[0].select('div.ChampionName')[0].stripped_strings)
```




    ['Orianna']



- `stripped_strings`을 사용하면 불필요한 문자는 지워지고 리스트로 나타난다.
