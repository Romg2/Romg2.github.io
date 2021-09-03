---
title: "[OPGG] 라이엇 API 사용하기"
excerpt: "데이터 분석가 양성 과정 1주차"
categories: 
  - OPGG
tags: 
    - Python
    - OPGG
toc: TRUE
toc_sticky: TRUE
use_math: TRUE
---

# 1. 내전 게임 불러오기


```python
import pandas as pd
import numpy as np
import requests
import matplotlib as plt

# Riot Developer Portal에서 받은 API KEY입니다.
# 해당 값을 포함한 모든 종류의 KEY는 코드에 직접적으로 노출되지 않도록 하는 것이 좋습니다.
api_key = 'RGAPI-0c612bd4-12a0-4358-8d6b-9ea0cd92b1cf'

# 방금 진행한 내전의 gameId값입니다. 클라이언트 대전기록에서 확인할 수 있습니다.
gameid = 5382324388
requesturl = "https://kr.api.riotgames.com/lol/match/v4/matches/"+str(gameid)+"?api_key="+api_key
r = requests.get(requesturl)
```

- [라이엇 개발자 홈페이지](https://developer.riotgames.com/)에서 API 탭에 가면 여러 자료를 불러올 수 있다.


- 이번엔 그 중에서 강의 중 내전한 자료를 불러올 것이다.


- gameid는 롤 대전기록 검색시 URL에서 확인 가능하다.


- API KEY는 24시간 정도 유지되며 그 이후 사용시 재발급하여 사용하면 된다.


```python
# 불러온 데이터 확인
# teamid 100: blue, 200: red
r.json().keys()
```




    dict_keys(['gameId', 'platformId', 'gameCreation', 'gameDuration', 'queueId', 'mapId', 'seasonId', 'gameVersion', 'gameMode', 'gameType', 'teams', 'participants', 'participantIdentities'])



- 불러온 데이터는 json 형식의 파일로 이루어져 있다.


- 각 key별로 어떤 자료가 있는지는 홈페이지에 자세한 설명이 있다.


```python
# 불러온 데이터 중 소환사1의 데이터 확인
# r.json()['participants'][0]
```

- participants에는 플레이한 소환사의 정보가 담겨있다.


```python
# DataFrame으로 변환해서 확인
pd.DataFrame(r.json()['participants'])
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
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>stats</th>
      <th>timeline</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>222</td>
      <td>7</td>
      <td>4</td>
      <td>{'participantId': 1, 'win': False, 'item0': 10...</td>
      <td>{'participantId': 1, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>234</td>
      <td>4</td>
      <td>11</td>
      <td>{'participantId': 2, 'win': False, 'item0': 31...</td>
      <td>{'participantId': 2, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>143</td>
      <td>4</td>
      <td>14</td>
      <td>{'participantId': 3, 'win': False, 'item0': 66...</td>
      <td>{'participantId': 3, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>100</td>
      <td>34</td>
      <td>12</td>
      <td>4</td>
      <td>{'participantId': 4, 'win': False, 'item0': 30...</td>
      <td>{'participantId': 4, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>58</td>
      <td>4</td>
      <td>14</td>
      <td>{'participantId': 5, 'win': False, 'item0': 66...</td>
      <td>{'participantId': 5, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>200</td>
      <td>48</td>
      <td>12</td>
      <td>4</td>
      <td>{'participantId': 6, 'win': True, 'item0': 304...</td>
      <td>{'participantId': 6, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>200</td>
      <td>84</td>
      <td>14</td>
      <td>4</td>
      <td>{'participantId': 7, 'win': True, 'item0': 105...</td>
      <td>{'participantId': 7, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>200</td>
      <td>43</td>
      <td>14</td>
      <td>4</td>
      <td>{'participantId': 8, 'win': True, 'item0': 385...</td>
      <td>{'participantId': 8, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>200</td>
      <td>203</td>
      <td>11</td>
      <td>4</td>
      <td>{'participantId': 9, 'win': True, 'item0': 313...</td>
      <td>{'participantId': 9, 'creepsPerMinDeltas': {'1...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>498</td>
      <td>7</td>
      <td>4</td>
      <td>{'participantId': 10, 'win': True, 'item0': 10...</td>
      <td>{'participantId': 10, 'creepsPerMinDeltas': {'...</td>
    </tr>
  </tbody>
</table>
</div>



- 이를 데이터프레임으로 바꿔보면 앞서 지정한 특정 게임의 플레이어 10명에 대한 정보가 담겨있다.


- 다만 stats와 timeline 등은 nested되어 있다.


```python
# nested된 부분 unnest해서 DataFrame으로 변환
df = pd.json_normalize(r.json()['participants'])
df.head()
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
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>stats.participantId</th>
      <th>stats.win</th>
      <th>stats.item0</th>
      <th>stats.item1</th>
      <th>stats.item2</th>
      <th>...</th>
      <th>timeline.csDiffPerMinDeltas.10-20</th>
      <th>timeline.csDiffPerMinDeltas.0-10</th>
      <th>timeline.xpDiffPerMinDeltas.10-20</th>
      <th>timeline.xpDiffPerMinDeltas.0-10</th>
      <th>timeline.damageTakenPerMinDeltas.10-20</th>
      <th>timeline.damageTakenPerMinDeltas.0-10</th>
      <th>timeline.damageTakenDiffPerMinDeltas.10-20</th>
      <th>timeline.damageTakenDiffPerMinDeltas.0-10</th>
      <th>timeline.role</th>
      <th>timeline.lane</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>222</td>
      <td>7</td>
      <td>4</td>
      <td>1</td>
      <td>False</td>
      <td>1055</td>
      <td>6671</td>
      <td>3094</td>
      <td>...</td>
      <td>-1.1</td>
      <td>-0.65</td>
      <td>36.35</td>
      <td>-24.55</td>
      <td>638.7</td>
      <td>252.8</td>
      <td>64.95</td>
      <td>-4.7</td>
      <td>DUO_CARRY</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>234</td>
      <td>4</td>
      <td>11</td>
      <td>2</td>
      <td>False</td>
      <td>3153</td>
      <td>2031</td>
      <td>3047</td>
      <td>...</td>
      <td>-0.9</td>
      <td>0.20</td>
      <td>-77.30</td>
      <td>-97.40</td>
      <td>1354.3</td>
      <td>892.3</td>
      <td>294.40</td>
      <td>206.7</td>
      <td>NONE</td>
      <td>JUNGLE</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>143</td>
      <td>4</td>
      <td>14</td>
      <td>3</td>
      <td>False</td>
      <td>6653</td>
      <td>2421</td>
      <td>3853</td>
      <td>...</td>
      <td>-1.1</td>
      <td>-0.65</td>
      <td>36.35</td>
      <td>-24.55</td>
      <td>482.4</td>
      <td>224.7</td>
      <td>64.95</td>
      <td>-4.7</td>
      <td>DUO_SUPPORT</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>100</td>
      <td>34</td>
      <td>12</td>
      <td>4</td>
      <td>4</td>
      <td>False</td>
      <td>3040</td>
      <td>6653</td>
      <td>0</td>
      <td>...</td>
      <td>0.6</td>
      <td>0.30</td>
      <td>36.60</td>
      <td>-52.30</td>
      <td>546.4</td>
      <td>127.0</td>
      <td>43.10</td>
      <td>-189.6</td>
      <td>SOLO</td>
      <td>MIDDLE</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>58</td>
      <td>4</td>
      <td>14</td>
      <td>5</td>
      <td>False</td>
      <td>6630</td>
      <td>2031</td>
      <td>3075</td>
      <td>...</td>
      <td>-1.9</td>
      <td>-1.00</td>
      <td>-99.20</td>
      <td>4.30</td>
      <td>1157.8</td>
      <td>683.7</td>
      <td>18.90</td>
      <td>103.7</td>
      <td>SOLO</td>
      <td>TOP</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 126 columns</p>
</div>



- `pd.json_normalize()`를 사용하면 다음과 같이 원하는 형태로 생성 가능하다.


```python
# 확인할 변수
COLUMNS = ["participantId", "teamId", "championId", "spell1Id", "spell2Id", "stats.win",
           "stats.item0","stats.item1","stats.item2","stats.item3","stats.item4","stats.item5","stats.item6",
           "stats.kills","stats.deaths","stats.assists","stats.totalDamageDealt","stats.totalDamageTaken",
           "stats.goldEarned"]
```


```python
# 확인할 변수만 가져와서 새로운 DataFrame에 저장
df2 = df[COLUMNS]
```


```python
# Column 이름에 불필요한 'stats.'가 붙은 걸 삭제
df2.columns = df2.columns.str.replace('stats.', '', regex = True)
```


```python
# 최종 추출 형태 확인
df2
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
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>win</th>
      <th>item0</th>
      <th>item1</th>
      <th>item2</th>
      <th>item3</th>
      <th>item4</th>
      <th>item5</th>
      <th>item6</th>
      <th>kills</th>
      <th>deaths</th>
      <th>assists</th>
      <th>totalDamageDealt</th>
      <th>totalDamageTaken</th>
      <th>goldEarned</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>222</td>
      <td>7</td>
      <td>4</td>
      <td>False</td>
      <td>1055</td>
      <td>6671</td>
      <td>3094</td>
      <td>3006</td>
      <td>1038</td>
      <td>1037</td>
      <td>3363</td>
      <td>7</td>
      <td>6</td>
      <td>8</td>
      <td>85575</td>
      <td>16660</td>
      <td>9877</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>234</td>
      <td>4</td>
      <td>11</td>
      <td>False</td>
      <td>3153</td>
      <td>2031</td>
      <td>3047</td>
      <td>0</td>
      <td>0</td>
      <td>6632</td>
      <td>3364</td>
      <td>6</td>
      <td>9</td>
      <td>10</td>
      <td>84491</td>
      <td>32960</td>
      <td>9053</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>143</td>
      <td>4</td>
      <td>14</td>
      <td>False</td>
      <td>6653</td>
      <td>2421</td>
      <td>3853</td>
      <td>3020</td>
      <td>3191</td>
      <td>1028</td>
      <td>3364</td>
      <td>3</td>
      <td>6</td>
      <td>11</td>
      <td>37787</td>
      <td>12358</td>
      <td>7565</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>100</td>
      <td>34</td>
      <td>12</td>
      <td>4</td>
      <td>False</td>
      <td>3040</td>
      <td>6653</td>
      <td>0</td>
      <td>0</td>
      <td>3020</td>
      <td>0</td>
      <td>3364</td>
      <td>4</td>
      <td>5</td>
      <td>4</td>
      <td>101048</td>
      <td>14466</td>
      <td>8237</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>58</td>
      <td>4</td>
      <td>14</td>
      <td>False</td>
      <td>6630</td>
      <td>2031</td>
      <td>3075</td>
      <td>3047</td>
      <td>1054</td>
      <td>0</td>
      <td>3364</td>
      <td>5</td>
      <td>7</td>
      <td>2</td>
      <td>70397</td>
      <td>29663</td>
      <td>8345</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>200</td>
      <td>48</td>
      <td>12</td>
      <td>4</td>
      <td>True</td>
      <td>3047</td>
      <td>1054</td>
      <td>6632</td>
      <td>3051</td>
      <td>3742</td>
      <td>3076</td>
      <td>3340</td>
      <td>3</td>
      <td>5</td>
      <td>11</td>
      <td>107619</td>
      <td>27460</td>
      <td>10940</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>200</td>
      <td>84</td>
      <td>14</td>
      <td>4</td>
      <td>True</td>
      <td>1054</td>
      <td>3020</td>
      <td>3157</td>
      <td>1052</td>
      <td>4633</td>
      <td>0</td>
      <td>3340</td>
      <td>2</td>
      <td>2</td>
      <td>5</td>
      <td>64034</td>
      <td>13449</td>
      <td>8417</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>200</td>
      <td>43</td>
      <td>14</td>
      <td>4</td>
      <td>True</td>
      <td>3853</td>
      <td>2065</td>
      <td>2055</td>
      <td>3011</td>
      <td>3158</td>
      <td>0</td>
      <td>3364</td>
      <td>0</td>
      <td>4</td>
      <td>18</td>
      <td>22255</td>
      <td>9783</td>
      <td>7729</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>200</td>
      <td>203</td>
      <td>11</td>
      <td>4</td>
      <td>True</td>
      <td>3139</td>
      <td>3026</td>
      <td>3006</td>
      <td>6672</td>
      <td>6676</td>
      <td>3094</td>
      <td>3364</td>
      <td>23</td>
      <td>7</td>
      <td>7</td>
      <td>200929</td>
      <td>24914</td>
      <td>18384</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>498</td>
      <td>7</td>
      <td>4</td>
      <td>True</td>
      <td>1055</td>
      <td>6671</td>
      <td>3508</td>
      <td>1038</td>
      <td>0</td>
      <td>3006</td>
      <td>3363</td>
      <td>5</td>
      <td>7</td>
      <td>7</td>
      <td>116952</td>
      <td>13582</td>
      <td>10475</td>
    </tr>
  </tbody>
</table>
</div>



- 각 플레이어의 팀, 선택한 챔피언, 스펠, 승리 여부 등 일부 정보만 불러왔다.


```python
# KDA = (K + A) / D
# GPM = goldEarned / (gameDuration_In_Munite)
# DPM = damageDealt / (gameDuration_In_Munite)
# DTPM = damageTaken / (gameDuration_In_Munite)
df3 = df2.assign(KDA=lambda x: (x['kills']+x['assists'])/x['deaths'])
df3['GPM'] = df3['goldEarned'] / r.json()['gameDuration'] * 60 # 분 단위
df3['DPM'] = df3['totalDamageDealt'] / r.json()['gameDuration'] * 60
df3['DTPM'] = df3['totalDamageTaken'] / r.json()['gameDuration'] * 60
```

- KDA, GPM 등 개인 지표를 추가하였다.


- KDA의 경우 Death가 0인 경우를 고려해서 짜도 될 것이다.


```python
df3
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
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>win</th>
      <th>item0</th>
      <th>item1</th>
      <th>item2</th>
      <th>item3</th>
      <th>...</th>
      <th>kills</th>
      <th>deaths</th>
      <th>assists</th>
      <th>totalDamageDealt</th>
      <th>totalDamageTaken</th>
      <th>goldEarned</th>
      <th>KDA</th>
      <th>GPM</th>
      <th>DPM</th>
      <th>DTPM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>222</td>
      <td>7</td>
      <td>4</td>
      <td>False</td>
      <td>1055</td>
      <td>6671</td>
      <td>3094</td>
      <td>3006</td>
      <td>...</td>
      <td>7</td>
      <td>6</td>
      <td>8</td>
      <td>85575</td>
      <td>16660</td>
      <td>9877</td>
      <td>2.500000</td>
      <td>368.086957</td>
      <td>3189.130435</td>
      <td>620.869565</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>234</td>
      <td>4</td>
      <td>11</td>
      <td>False</td>
      <td>3153</td>
      <td>2031</td>
      <td>3047</td>
      <td>0</td>
      <td>...</td>
      <td>6</td>
      <td>9</td>
      <td>10</td>
      <td>84491</td>
      <td>32960</td>
      <td>9053</td>
      <td>1.777778</td>
      <td>337.378882</td>
      <td>3148.732919</td>
      <td>1228.322981</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>143</td>
      <td>4</td>
      <td>14</td>
      <td>False</td>
      <td>6653</td>
      <td>2421</td>
      <td>3853</td>
      <td>3020</td>
      <td>...</td>
      <td>3</td>
      <td>6</td>
      <td>11</td>
      <td>37787</td>
      <td>12358</td>
      <td>7565</td>
      <td>2.333333</td>
      <td>281.925466</td>
      <td>1408.211180</td>
      <td>460.546584</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>100</td>
      <td>34</td>
      <td>12</td>
      <td>4</td>
      <td>False</td>
      <td>3040</td>
      <td>6653</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>4</td>
      <td>5</td>
      <td>4</td>
      <td>101048</td>
      <td>14466</td>
      <td>8237</td>
      <td>1.600000</td>
      <td>306.968944</td>
      <td>3765.763975</td>
      <td>539.105590</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>58</td>
      <td>4</td>
      <td>14</td>
      <td>False</td>
      <td>6630</td>
      <td>2031</td>
      <td>3075</td>
      <td>3047</td>
      <td>...</td>
      <td>5</td>
      <td>7</td>
      <td>2</td>
      <td>70397</td>
      <td>29663</td>
      <td>8345</td>
      <td>1.000000</td>
      <td>310.993789</td>
      <td>2623.490683</td>
      <td>1105.453416</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>200</td>
      <td>48</td>
      <td>12</td>
      <td>4</td>
      <td>True</td>
      <td>3047</td>
      <td>1054</td>
      <td>6632</td>
      <td>3051</td>
      <td>...</td>
      <td>3</td>
      <td>5</td>
      <td>11</td>
      <td>107619</td>
      <td>27460</td>
      <td>10940</td>
      <td>2.800000</td>
      <td>407.701863</td>
      <td>4010.645963</td>
      <td>1023.354037</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>200</td>
      <td>84</td>
      <td>14</td>
      <td>4</td>
      <td>True</td>
      <td>1054</td>
      <td>3020</td>
      <td>3157</td>
      <td>1052</td>
      <td>...</td>
      <td>2</td>
      <td>2</td>
      <td>5</td>
      <td>64034</td>
      <td>13449</td>
      <td>8417</td>
      <td>3.500000</td>
      <td>313.677019</td>
      <td>2386.360248</td>
      <td>501.204969</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>200</td>
      <td>43</td>
      <td>14</td>
      <td>4</td>
      <td>True</td>
      <td>3853</td>
      <td>2065</td>
      <td>2055</td>
      <td>3011</td>
      <td>...</td>
      <td>0</td>
      <td>4</td>
      <td>18</td>
      <td>22255</td>
      <td>9783</td>
      <td>7729</td>
      <td>4.500000</td>
      <td>288.037267</td>
      <td>829.378882</td>
      <td>364.583851</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>200</td>
      <td>203</td>
      <td>11</td>
      <td>4</td>
      <td>True</td>
      <td>3139</td>
      <td>3026</td>
      <td>3006</td>
      <td>6672</td>
      <td>...</td>
      <td>23</td>
      <td>7</td>
      <td>7</td>
      <td>200929</td>
      <td>24914</td>
      <td>18384</td>
      <td>4.285714</td>
      <td>685.118012</td>
      <td>7488.037267</td>
      <td>928.472050</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>498</td>
      <td>7</td>
      <td>4</td>
      <td>True</td>
      <td>1055</td>
      <td>6671</td>
      <td>3508</td>
      <td>1038</td>
      <td>...</td>
      <td>5</td>
      <td>7</td>
      <td>7</td>
      <td>116952</td>
      <td>13582</td>
      <td>10475</td>
      <td>1.714286</td>
      <td>390.372671</td>
      <td>4358.459627</td>
      <td>506.161491</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 23 columns</p>
</div>




```python
# 숫자로 된 각종 Key값에 대한 Value 가져오기
champion_constant = requests.get("http://ddragon.leagueoflegends.com/cdn/11.15.1/data/ko_KR/champion.json")
spell_constant = requests.get("http://ddragon.leagueoflegends.com/cdn/11.15.1/data/ko_KR/summoner.json")
item_constant = requests.get("http://ddragon.leagueoflegends.com/cdn/11.15.1/data/ko_KR/item.json")
```

- 챔피언, 스펠, 아이템에 대한 json 파일을 불러왔다.


- 이 역시 홈페이지에서 해당 URL을 확인 가능하다.


- 수업 당시 기준 11.15.1 버전 URL이나 업데이트 된다면 이를 바꿔주면 된다.


```python
# champion_constant.json()
```


```python
# Json 파일을 DataFrame으로 변환
champion_df = pd.DataFrame(champion_constant.json()['data']).T[['key','name']]
spell_df = pd.DataFrame(spell_constant.json()['data']).T[['key','name']]
item_df = pd.DataFrame(item_constant.json()['data']).T['name'].rename_axis("key").reset_index()

# 변수형 문자 -> 숫자
champion_df['key'] = pd.to_numeric(champion_df['key'])
spell_df['key'] = pd.to_numeric(spell_df['key'])
item_df['key'] = pd.to_numeric(item_df['key'])
```

- 이번에도 데이터 프레임 형태로 바꿔주되 각 파일의 컬럼이 매우 많아 일부만 사용한다.


```python
# 매칭되는 곳으로 내용 추가하여 새로운 DataFrame으로 저장
df4 = pd.merge(df3,champion_df,how = 'left', left_on = 'championId', right_on = 'key')
```


```python
# DataFrame 확인
df4
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
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>win</th>
      <th>item0</th>
      <th>item1</th>
      <th>item2</th>
      <th>item3</th>
      <th>...</th>
      <th>assists</th>
      <th>totalDamageDealt</th>
      <th>totalDamageTaken</th>
      <th>goldEarned</th>
      <th>KDA</th>
      <th>GPM</th>
      <th>DPM</th>
      <th>DTPM</th>
      <th>key</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>222</td>
      <td>7</td>
      <td>4</td>
      <td>False</td>
      <td>1055</td>
      <td>6671</td>
      <td>3094</td>
      <td>3006</td>
      <td>...</td>
      <td>8</td>
      <td>85575</td>
      <td>16660</td>
      <td>9877</td>
      <td>2.500000</td>
      <td>368.086957</td>
      <td>3189.130435</td>
      <td>620.869565</td>
      <td>222</td>
      <td>징크스</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>234</td>
      <td>4</td>
      <td>11</td>
      <td>False</td>
      <td>3153</td>
      <td>2031</td>
      <td>3047</td>
      <td>0</td>
      <td>...</td>
      <td>10</td>
      <td>84491</td>
      <td>32960</td>
      <td>9053</td>
      <td>1.777778</td>
      <td>337.378882</td>
      <td>3148.732919</td>
      <td>1228.322981</td>
      <td>234</td>
      <td>비에고</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>143</td>
      <td>4</td>
      <td>14</td>
      <td>False</td>
      <td>6653</td>
      <td>2421</td>
      <td>3853</td>
      <td>3020</td>
      <td>...</td>
      <td>11</td>
      <td>37787</td>
      <td>12358</td>
      <td>7565</td>
      <td>2.333333</td>
      <td>281.925466</td>
      <td>1408.211180</td>
      <td>460.546584</td>
      <td>143</td>
      <td>자이라</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>100</td>
      <td>34</td>
      <td>12</td>
      <td>4</td>
      <td>False</td>
      <td>3040</td>
      <td>6653</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>4</td>
      <td>101048</td>
      <td>14466</td>
      <td>8237</td>
      <td>1.600000</td>
      <td>306.968944</td>
      <td>3765.763975</td>
      <td>539.105590</td>
      <td>34</td>
      <td>애니비아</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>58</td>
      <td>4</td>
      <td>14</td>
      <td>False</td>
      <td>6630</td>
      <td>2031</td>
      <td>3075</td>
      <td>3047</td>
      <td>...</td>
      <td>2</td>
      <td>70397</td>
      <td>29663</td>
      <td>8345</td>
      <td>1.000000</td>
      <td>310.993789</td>
      <td>2623.490683</td>
      <td>1105.453416</td>
      <td>58</td>
      <td>레넥톤</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>200</td>
      <td>48</td>
      <td>12</td>
      <td>4</td>
      <td>True</td>
      <td>3047</td>
      <td>1054</td>
      <td>6632</td>
      <td>3051</td>
      <td>...</td>
      <td>11</td>
      <td>107619</td>
      <td>27460</td>
      <td>10940</td>
      <td>2.800000</td>
      <td>407.701863</td>
      <td>4010.645963</td>
      <td>1023.354037</td>
      <td>48</td>
      <td>트런들</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>200</td>
      <td>84</td>
      <td>14</td>
      <td>4</td>
      <td>True</td>
      <td>1054</td>
      <td>3020</td>
      <td>3157</td>
      <td>1052</td>
      <td>...</td>
      <td>5</td>
      <td>64034</td>
      <td>13449</td>
      <td>8417</td>
      <td>3.500000</td>
      <td>313.677019</td>
      <td>2386.360248</td>
      <td>501.204969</td>
      <td>84</td>
      <td>아칼리</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>200</td>
      <td>43</td>
      <td>14</td>
      <td>4</td>
      <td>True</td>
      <td>3853</td>
      <td>2065</td>
      <td>2055</td>
      <td>3011</td>
      <td>...</td>
      <td>18</td>
      <td>22255</td>
      <td>9783</td>
      <td>7729</td>
      <td>4.500000</td>
      <td>288.037267</td>
      <td>829.378882</td>
      <td>364.583851</td>
      <td>43</td>
      <td>카르마</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>200</td>
      <td>203</td>
      <td>11</td>
      <td>4</td>
      <td>True</td>
      <td>3139</td>
      <td>3026</td>
      <td>3006</td>
      <td>6672</td>
      <td>...</td>
      <td>7</td>
      <td>200929</td>
      <td>24914</td>
      <td>18384</td>
      <td>4.285714</td>
      <td>685.118012</td>
      <td>7488.037267</td>
      <td>928.472050</td>
      <td>203</td>
      <td>킨드레드</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>498</td>
      <td>7</td>
      <td>4</td>
      <td>True</td>
      <td>1055</td>
      <td>6671</td>
      <td>3508</td>
      <td>1038</td>
      <td>...</td>
      <td>7</td>
      <td>116952</td>
      <td>13582</td>
      <td>10475</td>
      <td>1.714286</td>
      <td>390.372671</td>
      <td>4358.459627</td>
      <td>506.161491</td>
      <td>498</td>
      <td>자야</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 25 columns</p>
</div>



- 챔피언 id를 기준으로 각 id가 어떤 챔피언인지 알 수 있게 merge한 결과이다.


```python
# 중복되는 값 삭제
df4 = df4.rename(columns={'name': 'Champion'}).drop(['championId','key'],axis = 1)
```


```python
df4 = pd.merge(df4,spell_df,how = 'left', left_on = 'spell1Id', right_on = 'key')
df4 = df4.rename(columns={'name': 'D'}).drop(['spell1Id','key'],axis = 1)
df4 = pd.merge(df4,spell_df,how = 'left', left_on = 'spell2Id', right_on = 'key')
df4 = df4.rename(columns={'name': 'F'}).drop(['spell2Id','key'],axis = 1)
```


```python
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item0', right_on = 'key')
df4 = df4.rename(columns={'name': 'Item1'}).drop(['item0','key'],axis = 1)
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item1', right_on = 'key')
df4 = df4.rename(columns={'name': 'Item2'}).drop(['item1','key'],axis = 1)
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item2', right_on = 'key')
df4 = df4.rename(columns={'name': 'Item3'}).drop(['item2','key'],axis = 1)
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item3', right_on = 'key')
df4 = df4.rename(columns={'name': 'Item4'}).drop(['item3','key'],axis = 1)
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item4', right_on = 'key')
df4 = df4.rename(columns={'name': 'Item5'}).drop(['item4','key'],axis = 1)
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item5', right_on = 'key')
df4 = df4.rename(columns={'name': 'Item6'}).drop(['item5','key'],axis = 1)
df4 = pd.merge(df4,item_df,how = 'left', left_on = 'item6', right_on = 'key')
df4 = df4.rename(columns={'name': 'Trinket'}).drop(['item6','key'],axis = 1)
```

- 같은 방법으로 스펠과 아이템의 이름을 확인하기 위해 merge한다.


```python
# 확인할 변수
COLUMNS_NEW = ["participantId", "teamId","win", "Champion", "D", "F", "KDA",
           "kills","deaths","assists","Item1","Item2","Item3","Item4","Item5","Item6","Trinket","GPM","DPM","DTPM"]
```


```python
df5 = df4[COLUMNS_NEW]
```


```python
df5
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
      <th>participantId</th>
      <th>teamId</th>
      <th>win</th>
      <th>Champion</th>
      <th>D</th>
      <th>F</th>
      <th>KDA</th>
      <th>kills</th>
      <th>deaths</th>
      <th>assists</th>
      <th>Item1</th>
      <th>Item2</th>
      <th>Item3</th>
      <th>Item4</th>
      <th>Item5</th>
      <th>Item6</th>
      <th>Trinket</th>
      <th>GPM</th>
      <th>DPM</th>
      <th>DTPM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>False</td>
      <td>징크스</td>
      <td>회복</td>
      <td>점멸</td>
      <td>2.500000</td>
      <td>7</td>
      <td>6</td>
      <td>8</td>
      <td>도란의 검</td>
      <td>돌풍</td>
      <td>고속 연사포</td>
      <td>광전사의 군화</td>
      <td>B.F. 대검</td>
      <td>곡괭이</td>
      <td>망원형 개조</td>
      <td>368.086957</td>
      <td>3189.130435</td>
      <td>620.869565</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>False</td>
      <td>비에고</td>
      <td>점멸</td>
      <td>강타</td>
      <td>1.777778</td>
      <td>6</td>
      <td>9</td>
      <td>10</td>
      <td>몰락한 왕의 검</td>
      <td>충전형 물약</td>
      <td>판금 장화</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>신성한 파괴자</td>
      <td>예언자의 렌즈</td>
      <td>337.378882</td>
      <td>3148.732919</td>
      <td>1228.322981</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>False</td>
      <td>자이라</td>
      <td>점멸</td>
      <td>점화</td>
      <td>2.333333</td>
      <td>3</td>
      <td>6</td>
      <td>11</td>
      <td>리안드리의 고뇌</td>
      <td>망가진 초시계</td>
      <td>얼음 정수의 파편</td>
      <td>마법사의 신발</td>
      <td>추적자의 팔목 보호대</td>
      <td>루비 수정</td>
      <td>예언자의 렌즈</td>
      <td>281.925466</td>
      <td>1408.211180</td>
      <td>460.546584</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>100</td>
      <td>False</td>
      <td>애니비아</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>1.600000</td>
      <td>4</td>
      <td>5</td>
      <td>4</td>
      <td>대천사의 포옹</td>
      <td>리안드리의 고뇌</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>마법사의 신발</td>
      <td>NaN</td>
      <td>예언자의 렌즈</td>
      <td>306.968944</td>
      <td>3765.763975</td>
      <td>539.105590</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>False</td>
      <td>레넥톤</td>
      <td>점멸</td>
      <td>점화</td>
      <td>1.000000</td>
      <td>5</td>
      <td>7</td>
      <td>2</td>
      <td>선혈포식자</td>
      <td>충전형 물약</td>
      <td>가시 갑옷</td>
      <td>판금 장화</td>
      <td>도란의 방패</td>
      <td>NaN</td>
      <td>예언자의 렌즈</td>
      <td>310.993789</td>
      <td>2623.490683</td>
      <td>1105.453416</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>200</td>
      <td>True</td>
      <td>트런들</td>
      <td>순간이동</td>
      <td>점멸</td>
      <td>2.800000</td>
      <td>3</td>
      <td>5</td>
      <td>11</td>
      <td>판금 장화</td>
      <td>도란의 방패</td>
      <td>신성한 파괴자</td>
      <td>온기가 필요한 자의 도끼</td>
      <td>망자의 갑옷</td>
      <td>덤불 조끼</td>
      <td>투명 와드</td>
      <td>407.701863</td>
      <td>4010.645963</td>
      <td>1023.354037</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>200</td>
      <td>True</td>
      <td>아칼리</td>
      <td>점화</td>
      <td>점멸</td>
      <td>3.500000</td>
      <td>2</td>
      <td>2</td>
      <td>5</td>
      <td>도란의 방패</td>
      <td>마법사의 신발</td>
      <td>존야의 모래시계</td>
      <td>증폭의 고서</td>
      <td>균열 생성기</td>
      <td>NaN</td>
      <td>투명 와드</td>
      <td>313.677019</td>
      <td>2386.360248</td>
      <td>501.204969</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>200</td>
      <td>True</td>
      <td>카르마</td>
      <td>점화</td>
      <td>점멸</td>
      <td>4.500000</td>
      <td>0</td>
      <td>4</td>
      <td>18</td>
      <td>얼음 정수의 파편</td>
      <td>슈렐리아의 군가</td>
      <td>제어 와드</td>
      <td>화학공학 부패기</td>
      <td>명석함의 아이오니아 장화</td>
      <td>NaN</td>
      <td>예언자의 렌즈</td>
      <td>288.037267</td>
      <td>829.378882</td>
      <td>364.583851</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>200</td>
      <td>True</td>
      <td>킨드레드</td>
      <td>강타</td>
      <td>점멸</td>
      <td>4.285714</td>
      <td>23</td>
      <td>7</td>
      <td>7</td>
      <td>헤르메스의 시미터</td>
      <td>수호 천사</td>
      <td>광전사의 군화</td>
      <td>크라켄 학살자</td>
      <td>징수의 총</td>
      <td>고속 연사포</td>
      <td>예언자의 렌즈</td>
      <td>685.118012</td>
      <td>7488.037267</td>
      <td>928.472050</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>True</td>
      <td>자야</td>
      <td>회복</td>
      <td>점멸</td>
      <td>1.714286</td>
      <td>5</td>
      <td>7</td>
      <td>7</td>
      <td>도란의 검</td>
      <td>돌풍</td>
      <td>정수 약탈자</td>
      <td>B.F. 대검</td>
      <td>NaN</td>
      <td>광전사의 군화</td>
      <td>망원형 개조</td>
      <td>390.372671</td>
      <td>4358.459627</td>
      <td>506.161491</td>
    </tr>
  </tbody>
</table>
</div>



- 최종적으로 이 게임의 정보에 대해 위와 같이 확인 가능하다.


- 이 게임은 red팀이 승리하였고 킨드레드가 하드캐리한 것이 KDA, DPM 등을 통해 보인다.
