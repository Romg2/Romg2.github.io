---
title: "[OPGG] 각 리그별 팀 성적 시각화"
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

# 각 리그별 팀 성적

이번에도 웹 스크래핑 배운 내용을 토대로 원하는 자료를 가져와 볼 것이다.

여기선 각 롤 리그별로 팀들의 성적을 불러와 시각화 해보았다.


```python
import numpy as np
import pandas as pd
import requests
from bs4 import BeautifulSoup
from selenium import webdriver

import plotly.graph_objects as go
import plotly.figure_factory as ff
```


```python
def league_result(league):
    # 리그에 따른 URL 설정
    url = f"https://qwer.gg/leagues/{league}/2021/summer"

    # 리그 정보 테이블 HTML 사용
    QWER_html = requests.get(url).text
    QWER_soup = BeautifulSoup(QWER_html, "html.parser")
    info = QWER_soup.find("table", attrs= {"class":"BorderedTable RankTable Ranks--desktop"}).find("tbody").find_all("tr")

    # 빈 데이터 프레임
    data = pd.DataFrame()
    
    # 데이터 적재
    for i in range(len(info)):
        lst = []
        # 순위/팀
        rank = info[i].find("td", attrs= {"class":"RankTable__rank Gilroy"}).get_text()
        team = info[i].find("td", attrs= {"class":"RankTable__name Gilroy"}).find("span", attrs={"class":"hidden-in-desktop"}).get_text()

        # 매치 승/패/승률
        temp = info[i].find_all("td")[3].get_text().split()
        win = temp[0][:-1]
        lose = temp[1][:-1]
        # win_rate = win / (win+lose)

        # 세트 승/패/승률
        temp2 = info[i].find_all("td")[5].get_text()
        set_win = temp[0][:-1]
        set_lose = temp[1][:-1]

        point = info[i].find("td", attrs= {"class":"RankTable__point Gilroy"}).get_text()

        # 테이블 정보 append
        lst.append(rank) ; lst.append(team) ; lst.append(win) ; lst.append(lose)
        lst.append(set_win) ; lst.append(set_lose) ; lst.append(point)

        # 데이터 프레임
        temp_df = pd.DataFrame(lst)
        data = pd.concat([data, temp_df], axis=1)

    # 데이터 프레임 형태 설정
    data = data.T
    data.index = range(data.shape[0])
    data.columns = ["rank", "team", "win", "lose", "set_win", "set_lose", "point"]

    # 컬럼별 타입 설정
    data["rank"] = data["rank"].astype(int)
    data["win"] = data["win"].astype(int)
    data["lose"] = data["lose"].astype(int)
    data["set_win"] = data["set_win"].astype(int)
    data["set_lose"] = data["set_lose"].astype(int)
    data["point"] = data["point"].astype(int)

    # 시각화 작업
    # 테이블 생성
    fig = ff.create_table(data, height_constant=60)

    # 매치 승/패 막대 그래프
    trace1 = go.Bar(x=data["team"], y=data["win"], xaxis='x2', yaxis='y2',
                    marker=dict(color='#0099ff'),
                    name='Win')
    trace2 = go.Bar(x=data["team"], y=data["lose"], xaxis='x2', yaxis='y2',
                    marker=dict(color='#404040'),
                    name='lose')

    fig.add_traces([trace1, trace2])

    # axis 설정
    fig['layout']['xaxis2'] = {}
    fig['layout']['yaxis2'] = {}
    fig.layout.yaxis.update({'domain': [0, .45]})
    fig.layout.yaxis2.update({'domain': [.6, 1]})
    fig.layout.yaxis2.update({'anchor': 'x2'})
    fig.layout.xaxis2.update({'anchor': 'y2'})
    fig.layout.yaxis2.update({'title': 'Goals'})

    # layout 설정
    fig.layout.margin.update({'t':75, 'l':50})
    fig.layout.update({'title':f'2021 {league} Result'})
    fig.layout.update({'height':800})

    # plot 생성
    fig.write_html(f"{league}.html")
    return fig.show()
```

- 사실 각 태그들은 직접 확인하여야 하니까 따로 설명할 것이 없다.


```python
league = "LCK"
league_result(league)
```

{% include plotly/LCK.html %}


- 한국 LCK 리그의 팀별 정보를 테이블과 막대 그래프로 시각화 하였다.


- DRX는 18전 2승 16패로 패가 매우 많음이 확인된다.


```python
league = "LPL"
league_result(league)
```

{% include plotly/LPL.html %}


- 이번엔 중국 LPL 리그를 확인해보았다.


- LPL에선 V5라는 팀이 0승 16패로 전패팀이 존재한다.


```python
league = "LEC"
league_result(league)
```

{% include plotly/LEC.html %}



- 마지막으로 LEC를 확인하였다.
