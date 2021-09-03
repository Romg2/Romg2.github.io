---
title: "[OPGG] 롤 진영별 승률 확인하기"
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

# 1. 블루/레드 승률


```python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import pymysql
import sys
import os
```


```python
# # OP.GG Database에 접근하기 위한 연결 객체 생성
# con = pymysql.connect(
#     user = os.environ['LOL_KR_ID'],
#     passwd = os.environ['LOL_KR_PW'],
#     host = os.environ['LOL_KR_HOST'],
#     db = 'lol',
#     charset = 'utf8'
# )
# cursor = con.cursor(pymysql.cursors.DictCursor)
```


```python
# # 데이터 구조 파악을 위한 sample data fetch
# # SQL
# cursor.execute('''
# SELECT STRAIGHT_JOIN *
# FROM opGame o FORCE INDEX (ix_createDate),
# p_opGameStats p FORCE INDEX (`PRIMARY`)
# WHERE o.gameId = p.gameId
# AND o.createDate >= '2021-08-12'
# AND p.createDate >= '2021-08-12'
# ORDER BY o.createDate DESC
# LIMIT 30
# ''')
# sample = cursor.fetchall()

# # DataFrame으로 변환
# sample = pd.DataFrame(sample)
```


```python
# sample.to_csv("sample.csv", mode='w')
```


```python
# 미리 뽑아둔 CSV 파일에서 가져오기
sample = pd.read_csv('Day02_01_sample.csv').drop("Unnamed: 0", axis=1)
```

- 원래는 `pymysql` 패키지를 통해 OPGG 데이터베이스에서 자료를 불러온다.


- 나는 접근 권한이 없으므로 미리 만든 CSV파일을 제공받아 사용하였다.


```python
# 데이터 확인
sample.head()
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
      <th>gameId</th>
      <th>gameMapId</th>
      <th>gameType</th>
      <th>subType</th>
      <th>createDate</th>
      <th>gameLength</th>
      <th>interestScore</th>
      <th>goldLeadChangeCount</th>
      <th>isSurrender</th>
      <th>coreMinute</th>
      <th>...</th>
      <th>tierRank</th>
      <th>position</th>
      <th>isDetail</th>
      <th>isRanked</th>
      <th>perk0</th>
      <th>perkPrimaryStyle</th>
      <th>perkSubStyle</th>
      <th>opScore</th>
      <th>opScoreRank</th>
      <th>isOPScoreMaxInTeam</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5386357435</td>
      <td>11</td>
      <td>MATCHED_GAME</td>
      <td>420</td>
      <td>2021-08-12 15:53:21</td>
      <td>1859</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>S220</td>
      <td>T</td>
      <td>1</td>
      <td>1</td>
      <td>8229</td>
      <td>8200</td>
      <td>8000</td>
      <td>5.17</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5386357435</td>
      <td>11</td>
      <td>MATCHED_GAME</td>
      <td>420</td>
      <td>2021-08-12 15:53:21</td>
      <td>1859</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>S271</td>
      <td>S</td>
      <td>1</td>
      <td>1</td>
      <td>8437</td>
      <td>8400</td>
      <td>8000</td>
      <td>6.32</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5386357435</td>
      <td>11</td>
      <td>MATCHED_GAME</td>
      <td>420</td>
      <td>2021-08-12 15:53:21</td>
      <td>1859</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>S247</td>
      <td>M</td>
      <td>1</td>
      <td>1</td>
      <td>8214</td>
      <td>8200</td>
      <td>8300</td>
      <td>8.14</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5386357435</td>
      <td>11</td>
      <td>MATCHED_GAME</td>
      <td>420</td>
      <td>2021-08-12 15:53:21</td>
      <td>1859</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>S222</td>
      <td>A</td>
      <td>1</td>
      <td>1</td>
      <td>8128</td>
      <td>8100</td>
      <td>8000</td>
      <td>5.44</td>
      <td>7</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5386357435</td>
      <td>11</td>
      <td>MATCHED_GAME</td>
      <td>420</td>
      <td>2021-08-12 15:53:21</td>
      <td>1859</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>S216</td>
      <td>A</td>
      <td>1</td>
      <td>1</td>
      <td>8229</td>
      <td>8200</td>
      <td>8300</td>
      <td>6.85</td>
      <td>4</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 87 columns</p>
</div>




```python
# columns 확인
print(sample.columns)
```

    Index(['gameId', 'gameMapId', 'gameType', 'subType', 'createDate',
           'gameLength', 'interestScore', 'goldLeadChangeCount', 'isSurrender',
           'coreMinute', 'coreMinuteGoldDiff', 'isComebackWin',
           'maxGoldDiffBeforeComeback', 'maxGoldDiffMinBeforeComeback',
           'firstGoldLeadMinAfterComeback', 'version', 'scanned', 'isFullData',
           'isPlusData', 'p.gameId', 'p.createDate', 'teamId', 'summonerId',
           'isUnknownSummoner', 'participantId', 'championId', 'result',
           'skinIndex', 'spell1', 'spell2', 'leaver', 'experienceEarned',
           'eligibleFirstWinOfDay', 'ipEarned', 'boostXpEarned', 'boostIpEarned',
           'premadeSize', 'item0', 'item1', 'item2', 'item3', 'item4', 'item5',
           'item6', 'level', 'championsKilled', 'numDeaths', 'assists',
           'neutralMinionsKilled', 'turretsKilled', 'barracksKilled',
           'minionsKilled', 'largestMultiKill', 'largestCriticalStrike',
           'largestKillingSpree', 'goldEarned', 'physicalDamageDealtToChampions',
           'magicDamageDealtPlayer', 'physicalDamageTaken',
           'sightWardsBoughtInGame', 'visionWardsBoughtInGame', 'wardKilled',
           'wardPlaced', 'totalHeal', 'totalDamageDealtToChampions',
           'totalDamageDealt', 'totalDamageTaken',
           'neutralMinionsKilledEnemyJungle', 'neutralMinionsKilledTeamJungle',
           'visionScore', 'timeCCingOthers', 'damageSelfMitigated',
           'damageDealtToObjectives', 'damageDealtToTurrets', 'lane', 'role',
           'keystoneMasteryId', 'tierRank', 'position', 'isDetail', 'isRanked',
           'perk0', 'perkPrimaryStyle', 'perkSubStyle', 'opScore', 'opScoreRank',
           'isOPScoreMaxInTeam'],
          dtype='object')
    


```python
# Column 수가 많아서 print했을 때 다 보이게 옵션 설정
pd.set_option('display.max_columns', None)

# 값이 정상적인지 보기 위해 describe로 요약 정보 출력
sample.describe(include='all',datetime_is_numeric=True)
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
      <th>gameId</th>
      <th>gameMapId</th>
      <th>gameType</th>
      <th>subType</th>
      <th>createDate</th>
      <th>gameLength</th>
      <th>interestScore</th>
      <th>goldLeadChangeCount</th>
      <th>isSurrender</th>
      <th>coreMinute</th>
      <th>coreMinuteGoldDiff</th>
      <th>isComebackWin</th>
      <th>maxGoldDiffBeforeComeback</th>
      <th>maxGoldDiffMinBeforeComeback</th>
      <th>firstGoldLeadMinAfterComeback</th>
      <th>version</th>
      <th>scanned</th>
      <th>isFullData</th>
      <th>isPlusData</th>
      <th>p.gameId</th>
      <th>p.createDate</th>
      <th>teamId</th>
      <th>summonerId</th>
      <th>isUnknownSummoner</th>
      <th>participantId</th>
      <th>championId</th>
      <th>result</th>
      <th>skinIndex</th>
      <th>spell1</th>
      <th>spell2</th>
      <th>leaver</th>
      <th>experienceEarned</th>
      <th>eligibleFirstWinOfDay</th>
      <th>ipEarned</th>
      <th>boostXpEarned</th>
      <th>boostIpEarned</th>
      <th>premadeSize</th>
      <th>item0</th>
      <th>item1</th>
      <th>item2</th>
      <th>item3</th>
      <th>item4</th>
      <th>item5</th>
      <th>item6</th>
      <th>level</th>
      <th>championsKilled</th>
      <th>numDeaths</th>
      <th>assists</th>
      <th>neutralMinionsKilled</th>
      <th>turretsKilled</th>
      <th>barracksKilled</th>
      <th>minionsKilled</th>
      <th>largestMultiKill</th>
      <th>largestCriticalStrike</th>
      <th>largestKillingSpree</th>
      <th>goldEarned</th>
      <th>physicalDamageDealtToChampions</th>
      <th>magicDamageDealtPlayer</th>
      <th>physicalDamageTaken</th>
      <th>sightWardsBoughtInGame</th>
      <th>visionWardsBoughtInGame</th>
      <th>wardKilled</th>
      <th>wardPlaced</th>
      <th>totalHeal</th>
      <th>totalDamageDealtToChampions</th>
      <th>totalDamageDealt</th>
      <th>totalDamageTaken</th>
      <th>neutralMinionsKilledEnemyJungle</th>
      <th>neutralMinionsKilledTeamJungle</th>
      <th>visionScore</th>
      <th>timeCCingOthers</th>
      <th>damageSelfMitigated</th>
      <th>damageDealtToObjectives</th>
      <th>damageDealtToTurrets</th>
      <th>lane</th>
      <th>role</th>
      <th>keystoneMasteryId</th>
      <th>tierRank</th>
      <th>position</th>
      <th>isDetail</th>
      <th>isRanked</th>
      <th>perk0</th>
      <th>perkPrimaryStyle</th>
      <th>perkSubStyle</th>
      <th>opScore</th>
      <th>opScoreRank</th>
      <th>isOPScoreMaxInTeam</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3.000000e+01</td>
      <td>30.000000</td>
      <td>30</td>
      <td>30.000000</td>
      <td>30</td>
      <td>30.000000</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>30.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>30</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>3.000000e+01</td>
      <td>30</td>
      <td>30.000000</td>
      <td>3.000000e+01</td>
      <td>30.0</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30</td>
      <td>30.0</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.0</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.0</td>
      <td>30.000000</td>
      <td>30.00000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30</td>
      <td>30</td>
      <td>30.0</td>
      <td>21</td>
      <td>20</td>
      <td>30.0</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5</td>
      <td>5</td>
      <td>NaN</td>
      <td>21</td>
      <td>5</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>top</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>MATCHED_GAME</td>
      <td>NaN</td>
      <td>2021-08-12 15:53:21</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>11.16.390.1945</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2021-08-12 15:53:21</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>LOSE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NONE</td>
      <td>DUO_SUPPOR</td>
      <td>NaN</td>
      <td>S220</td>
      <td>T</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>30</td>
      <td>NaN</td>
      <td>10</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>30</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>15</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>13</td>
      <td>NaN</td>
      <td>1</td>
      <td>4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.386362e+09</td>
      <td>11.333333</td>
      <td>NaN</td>
      <td>433.333333</td>
      <td>NaN</td>
      <td>1451.333333</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.386362e+09</td>
      <td>NaN</td>
      <td>150.000000</td>
      <td>4.941984e+07</td>
      <td>0.0</td>
      <td>5.500000</td>
      <td>149.333333</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>12.733333</td>
      <td>7.233333</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4478.066667</td>
      <td>3267.900000</td>
      <td>3547.700000</td>
      <td>3154.466667</td>
      <td>2350.333333</td>
      <td>1703.700000</td>
      <td>2850.966667</td>
      <td>15.000000</td>
      <td>7.200000</td>
      <td>7.200000</td>
      <td>12.033333</td>
      <td>19.200000</td>
      <td>0.700000</td>
      <td>0.133333</td>
      <td>86.766667</td>
      <td>1.666667</td>
      <td>159.866667</td>
      <td>2.600000</td>
      <td>11354.100000</td>
      <td>8714.566667</td>
      <td>9815.466667</td>
      <td>12254.333333</td>
      <td>0.0</td>
      <td>1.266667</td>
      <td>1.60000</td>
      <td>6.433333</td>
      <td>6539.466667</td>
      <td>19219.766667</td>
      <td>95026.933333</td>
      <td>23116.166667</td>
      <td>3.300000</td>
      <td>11.033333</td>
      <td>14.066667</td>
      <td>20.333333</td>
      <td>17038.466667</td>
      <td>6389.700000</td>
      <td>1977.466667</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>0.333333</td>
      <td>8124.466667</td>
      <td>8106.666667</td>
      <td>8183.333333</td>
      <td>4.248000</td>
      <td>3.666667</td>
      <td>0.133333</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.159648e+04</td>
      <td>0.479463</td>
      <td>NaN</td>
      <td>12.685407</td>
      <td>NaN</td>
      <td>321.383146</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.159648e+04</td>
      <td>NaN</td>
      <td>50.854763</td>
      <td>3.312891e+07</td>
      <td>0.0</td>
      <td>2.921384</td>
      <td>141.786224</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>8.885996</td>
      <td>8.605064</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1946.282078</td>
      <td>1716.157041</td>
      <td>1799.336221</td>
      <td>1379.155932</td>
      <td>1724.201371</td>
      <td>1689.720343</td>
      <td>807.671898</td>
      <td>1.982684</td>
      <td>4.029204</td>
      <td>2.696102</td>
      <td>8.015000</td>
      <td>40.958684</td>
      <td>1.087547</td>
      <td>0.345746</td>
      <td>69.981122</td>
      <td>0.758098</td>
      <td>382.097364</td>
      <td>1.693802</td>
      <td>2591.659878</td>
      <td>8109.920650</td>
      <td>9675.071080</td>
      <td>5854.532181</td>
      <td>0.0</td>
      <td>1.552158</td>
      <td>2.19089</td>
      <td>7.242991</td>
      <td>6187.051867</td>
      <td>6806.726332</td>
      <td>61450.746398</td>
      <td>8060.104013</td>
      <td>8.363096</td>
      <td>26.110486</td>
      <td>14.843862</td>
      <td>17.070510</td>
      <td>11376.787213</td>
      <td>10922.650235</td>
      <td>2687.352264</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.479463</td>
      <td>137.492503</td>
      <td>128.474694</td>
      <td>134.121235</td>
      <td>3.182913</td>
      <td>3.555795</td>
      <td>0.345746</td>
    </tr>
    <tr>
      <th>min</th>
      <td>5.386339e+09</td>
      <td>11.000000</td>
      <td>NaN</td>
      <td>420.000000</td>
      <td>NaN</td>
      <td>1089.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.386339e+09</td>
      <td>NaN</td>
      <td>100.000000</td>
      <td>1.228419e+06</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1054.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>11.000000</td>
      <td>1.000000</td>
      <td>3.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>5.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>6046.000000</td>
      <td>260.000000</td>
      <td>0.000000</td>
      <td>3156.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>329.000000</td>
      <td>6659.000000</td>
      <td>10532.000000</td>
      <td>7719.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3181.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>8005.000000</td>
      <td>8000.000000</td>
      <td>8000.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5.386339e+09</td>
      <td>11.000000</td>
      <td>NaN</td>
      <td>420.000000</td>
      <td>NaN</td>
      <td>1089.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.386339e+09</td>
      <td>NaN</td>
      <td>100.000000</td>
      <td>1.886260e+07</td>
      <td>0.0</td>
      <td>3.000000</td>
      <td>55.750000</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>7.000000</td>
      <td>4.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3068.000000</td>
      <td>2567.250000</td>
      <td>3043.250000</td>
      <td>3035.000000</td>
      <td>1052.250000</td>
      <td>0.000000</td>
      <td>2052.000000</td>
      <td>13.250000</td>
      <td>4.250000</td>
      <td>5.000000</td>
      <td>4.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>30.750000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>9661.750000</td>
      <td>1043.250000</td>
      <td>2015.500000</td>
      <td>7782.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>2098.250000</td>
      <td>15237.500000</td>
      <td>49813.000000</td>
      <td>17870.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>9.000000</td>
      <td>9202.250000</td>
      <td>497.750000</td>
      <td>294.250000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>8010.000000</td>
      <td>8000.000000</td>
      <td>8100.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>5.386357e+09</td>
      <td>11.000000</td>
      <td>NaN</td>
      <td>430.000000</td>
      <td>NaN</td>
      <td>1406.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.386357e+09</td>
      <td>NaN</td>
      <td>150.000000</td>
      <td>5.758059e+07</td>
      <td>0.0</td>
      <td>5.500000</td>
      <td>87.500000</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>12.000000</td>
      <td>4.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3854.000000</td>
      <td>3092.500000</td>
      <td>3095.000000</td>
      <td>3105.500000</td>
      <td>3020.000000</td>
      <td>1036.000000</td>
      <td>3340.000000</td>
      <td>15.500000</td>
      <td>6.000000</td>
      <td>8.000000</td>
      <td>10.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>56.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>11878.000000</td>
      <td>7584.500000</td>
      <td>6340.500000</td>
      <td>11663.000000</td>
      <td>0.0</td>
      <td>1.000000</td>
      <td>1.00000</td>
      <td>5.500000</td>
      <td>5214.500000</td>
      <td>18970.500000</td>
      <td>83774.000000</td>
      <td>23526.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>13.500000</td>
      <td>18.000000</td>
      <td>11953.000000</td>
      <td>2839.000000</td>
      <td>856.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>0.000000</td>
      <td>8120.000000</td>
      <td>8100.000000</td>
      <td>8200.000000</td>
      <td>5.435000</td>
      <td>3.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>5.386390e+09</td>
      <td>12.000000</td>
      <td>NaN</td>
      <td>450.000000</td>
      <td>NaN</td>
      <td>1859.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.386390e+09</td>
      <td>NaN</td>
      <td>200.000000</td>
      <td>8.357982e+07</td>
      <td>0.0</td>
      <td>8.000000</td>
      <td>217.250000</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>14.000000</td>
      <td>4.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6654.500000</td>
      <td>3157.750000</td>
      <td>3182.750000</td>
      <td>3157.750000</td>
      <td>3111.000000</td>
      <td>3082.250000</td>
      <td>3363.750000</td>
      <td>16.000000</td>
      <td>10.750000</td>
      <td>9.000000</td>
      <td>17.250000</td>
      <td>8.750000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>139.250000</td>
      <td>2.000000</td>
      <td>22.750000</td>
      <td>3.000000</td>
      <td>12981.250000</td>
      <td>14667.250000</td>
      <td>16026.750000</td>
      <td>15075.750000</td>
      <td>0.0</td>
      <td>2.000000</td>
      <td>2.00000</td>
      <td>9.750000</td>
      <td>8657.250000</td>
      <td>24286.250000</td>
      <td>117880.250000</td>
      <td>28370.500000</td>
      <td>0.750000</td>
      <td>4.000000</td>
      <td>21.750000</td>
      <td>27.500000</td>
      <td>22984.750000</td>
      <td>8188.750000</td>
      <td>2779.500000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>1.000000</td>
      <td>8214.000000</td>
      <td>8200.000000</td>
      <td>8300.000000</td>
      <td>6.925000</td>
      <td>6.750000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.386390e+09</td>
      <td>12.000000</td>
      <td>NaN</td>
      <td>450.000000</td>
      <td>NaN</td>
      <td>1859.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.386390e+09</td>
      <td>NaN</td>
      <td>200.000000</td>
      <td>8.818104e+07</td>
      <td>0.0</td>
      <td>10.000000</td>
      <td>517.000000</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>32.000000</td>
      <td>32.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6692.000000</td>
      <td>6653.000000</td>
      <td>6692.000000</td>
      <td>6694.000000</td>
      <td>6672.000000</td>
      <td>6632.000000</td>
      <td>3364.000000</td>
      <td>18.000000</td>
      <td>15.000000</td>
      <td>12.000000</td>
      <td>31.000000</td>
      <td>153.000000</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>271.000000</td>
      <td>4.000000</td>
      <td>1866.000000</td>
      <td>7.000000</td>
      <td>16862.000000</td>
      <td>27427.000000</td>
      <td>36284.000000</td>
      <td>33100.000000</td>
      <td>0.0</td>
      <td>6.000000</td>
      <td>7.00000</td>
      <td>27.000000</td>
      <td>26173.000000</td>
      <td>36544.000000</td>
      <td>251762.000000</td>
      <td>45985.000000</td>
      <td>32.000000</td>
      <td>103.000000</td>
      <td>58.000000</td>
      <td>76.000000</td>
      <td>47384.000000</td>
      <td>54606.000000</td>
      <td>11389.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>1.000000</td>
      <td>8439.000000</td>
      <td>8400.000000</td>
      <td>8400.000000</td>
      <td>8.140000</td>
      <td>10.000000</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



- 우선 컬럼이 매우 많다.


```python
# 필요한 값만 선택

# position은 opgg가 만든 ai
sample = sample[['gameId','subType','gameLength','teamId','championId','result','position']]
```

- 필요한 컬럼만 선택하였다.


- 여기서 subType은 솔랭, 자랭 등 게임 타입이며 position은 OPGG에서 딥러닝으로 만든 컬럼이라고 한다.


```python
# 최종적으로 만들어진 sample dataframe
sample
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
      <th>gameId</th>
      <th>subType</th>
      <th>gameLength</th>
      <th>teamId</th>
      <th>championId</th>
      <th>result</th>
      <th>position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>200</td>
      <td>74</td>
      <td>LOSE</td>
      <td>T</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>200</td>
      <td>235</td>
      <td>LOSE</td>
      <td>S</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>100</td>
      <td>90</td>
      <td>WIN</td>
      <td>M</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>200</td>
      <td>202</td>
      <td>LOSE</td>
      <td>A</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>100</td>
      <td>115</td>
      <td>WIN</td>
      <td>A</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>200</td>
      <td>234</td>
      <td>LOSE</td>
      <td>J</td>
    </tr>
    <tr>
      <th>6</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>100</td>
      <td>48</td>
      <td>WIN</td>
      <td>J</td>
    </tr>
    <tr>
      <th>7</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>200</td>
      <td>517</td>
      <td>LOSE</td>
      <td>M</td>
    </tr>
    <tr>
      <th>8</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>100</td>
      <td>43</td>
      <td>WIN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>9</th>
      <td>5386357435</td>
      <td>420</td>
      <td>1859</td>
      <td>100</td>
      <td>92</td>
      <td>WIN</td>
      <td>T</td>
    </tr>
    <tr>
      <th>10</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>100</td>
      <td>126</td>
      <td>LOSE</td>
      <td>T</td>
    </tr>
    <tr>
      <th>11</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>100</td>
      <td>517</td>
      <td>LOSE</td>
      <td>M</td>
    </tr>
    <tr>
      <th>12</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>200</td>
      <td>85</td>
      <td>WIN</td>
      <td>T</td>
    </tr>
    <tr>
      <th>13</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>200</td>
      <td>53</td>
      <td>WIN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>14</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>200</td>
      <td>203</td>
      <td>WIN</td>
      <td>J</td>
    </tr>
    <tr>
      <th>15</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>200</td>
      <td>166</td>
      <td>WIN</td>
      <td>M</td>
    </tr>
    <tr>
      <th>16</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>100</td>
      <td>64</td>
      <td>LOSE</td>
      <td>J</td>
    </tr>
    <tr>
      <th>17</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>200</td>
      <td>81</td>
      <td>WIN</td>
      <td>A</td>
    </tr>
    <tr>
      <th>18</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>100</td>
      <td>81</td>
      <td>LOSE</td>
      <td>A</td>
    </tr>
    <tr>
      <th>19</th>
      <td>5386338795</td>
      <td>430</td>
      <td>1406</td>
      <td>100</td>
      <td>350</td>
      <td>LOSE</td>
      <td>S</td>
    </tr>
    <tr>
      <th>20</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>200</td>
      <td>61</td>
      <td>LOSE</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>100</td>
      <td>17</td>
      <td>WIN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>100</td>
      <td>222</td>
      <td>WIN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>200</td>
      <td>8</td>
      <td>LOSE</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>100</td>
      <td>19</td>
      <td>WIN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>200</td>
      <td>54</td>
      <td>LOSE</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>200</td>
      <td>236</td>
      <td>LOSE</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>100</td>
      <td>3</td>
      <td>WIN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>100</td>
      <td>420</td>
      <td>WIN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>5386390166</td>
      <td>450</td>
      <td>1089</td>
      <td>200</td>
      <td>64</td>
      <td>LOSE</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



- subType 450은 칼바람이며 420은 솔랭, 430은 일반 게임이다.


- 보면 430은 일반 게임인데 포지션 분류가 되어 있는 것을 확인 할 수 있다.


- subType등에 대한 정보는 [여기](https://static.developer.riotgames.com/docs/lol/queues.json)를 참고하자.


- (API로 끌어온 데이터가 아닌 OPGG가 정제한 데이터베이스의 데이터이므로 컬럼명은 다를 수 있다.)


```python
# Reset sample data
%reset
```

    Once deleted, variables cannot be recovered. Proceed (y/[n])? y
    


```python
import numpy as np
import pandas as pd
from matplotlib import pyplot as plt
import pymysql
import sys
import os
from scipy.stats import norm
import requests
```


```python
# # OP.GG Database에 접근하기 위한 연결 객체 생성
# con = pymysql.connect(
#     user = os.environ['LOL_KR_ID'],
#     passwd = os.environ['LOL_KR_PW'],
#     host = os.environ['LOL_KR_HOST'],
#     db = 'lol',
#     charset = 'utf8'
# )
# cursor = con.cursor(pymysql.cursors.DictCursor)
```


```python
# # Full data fetch
# # SQL
# cursor.execute('''
# SELECT STRAIGHT_JOIN o.gameId, subType, gameLength, teamId, championId, result, position
# FROM opGame o FORCE INDEX (ix_createDate),
# p_opGameStats p FORCE INDEX (`PRIMARY`)
# WHERE o.gameId = p.gameId
# AND o.createDate >= '2021-08-12'
# AND p.createDate >= '2021-08-12'
# LIMIT 1000000                         
# ''')
# gamestats = cursor.fetchall()

# # DataFrame으로 변환
# gamestats = pd.DataFrame(gamestats)
```


```python
# gamestats.to_csv("gamestats.csv", mode='w')
```


```python
# 미리 뽑아둔 CSV 파일에서 가져오기
gamestats = pd.read_csv('Day02_01_gamestats.csv', index_col=0)
```

- 앞서 일부 샘플로 과정을 확인했으니 이제 전체 데이터로 진행해보자.


```python
# 데이터 확인
gamestats
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
      <th>gameId</th>
      <th>subType</th>
      <th>gameLength</th>
      <th>teamId</th>
      <th>championId</th>
      <th>result</th>
      <th>position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5385587730</td>
      <td>440</td>
      <td>1804</td>
      <td>200</td>
      <td>81</td>
      <td>WIN</td>
      <td>A</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5385587730</td>
      <td>440</td>
      <td>1804</td>
      <td>200</td>
      <td>107</td>
      <td>WIN</td>
      <td>J</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5385587730</td>
      <td>440</td>
      <td>1804</td>
      <td>100</td>
      <td>121</td>
      <td>LOSE</td>
      <td>J</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5385587730</td>
      <td>440</td>
      <td>1804</td>
      <td>100</td>
      <td>517</td>
      <td>LOSE</td>
      <td>M</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5385587730</td>
      <td>440</td>
      <td>1804</td>
      <td>100</td>
      <td>12</td>
      <td>LOSE</td>
      <td>S</td>
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
    </tr>
    <tr>
      <th>99995</th>
      <td>5385598894</td>
      <td>440</td>
      <td>1977</td>
      <td>200</td>
      <td>222</td>
      <td>WIN</td>
      <td>A</td>
    </tr>
    <tr>
      <th>99996</th>
      <td>5385598894</td>
      <td>440</td>
      <td>1977</td>
      <td>200</td>
      <td>234</td>
      <td>WIN</td>
      <td>J</td>
    </tr>
    <tr>
      <th>99997</th>
      <td>5385598894</td>
      <td>440</td>
      <td>1977</td>
      <td>100</td>
      <td>81</td>
      <td>LOSE</td>
      <td>A</td>
    </tr>
    <tr>
      <th>99998</th>
      <td>5385598894</td>
      <td>440</td>
      <td>1977</td>
      <td>100</td>
      <td>1</td>
      <td>LOSE</td>
      <td>M</td>
    </tr>
    <tr>
      <th>99999</th>
      <td>5385598894</td>
      <td>440</td>
      <td>1977</td>
      <td>100</td>
      <td>8</td>
      <td>LOSE</td>
      <td>T</td>
    </tr>
  </tbody>
</table>
<p>100000 rows × 7 columns</p>
</div>




```python
# 데이터 타입 확인
print(gamestats.dtypes)
```

    gameId         int64
    subType        int64
    gameLength     int64
    teamId         int64
    championId     int64
    result        object
    position      object
    dtype: object
    


```python
# 데이터 요약 정보
gamestats.describe(include='all',datetime_is_numeric=True)
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
      <th>gameId</th>
      <th>subType</th>
      <th>gameLength</th>
      <th>teamId</th>
      <th>championId</th>
      <th>result</th>
      <th>position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1.000000e+05</td>
      <td>100000.000000</td>
      <td>100000.000000</td>
      <td>100000.000000</td>
      <td>100000.000000</td>
      <td>100000</td>
      <td>66998</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3</td>
      <td>5</td>
    </tr>
    <tr>
      <th>top</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>WIN</td>
      <td>M</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>49677</td>
      <td>13400</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.385674e+09</td>
      <td>447.026200</td>
      <td>1491.257480</td>
      <td>149.990000</td>
      <td>149.647020</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.216472e+04</td>
      <td>111.647298</td>
      <td>489.932212</td>
      <td>50.000249</td>
      <td>178.131795</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>min</th>
      <td>5.385581e+09</td>
      <td>420.000000</td>
      <td>190.000000</td>
      <td>100.000000</td>
      <td>1.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5.385629e+09</td>
      <td>420.000000</td>
      <td>1141.000000</td>
      <td>100.000000</td>
      <td>43.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>5.385673e+09</td>
      <td>430.000000</td>
      <td>1440.000000</td>
      <td>100.000000</td>
      <td>86.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>5.385712e+09</td>
      <td>450.000000</td>
      <td>1826.000000</td>
      <td>200.000000</td>
      <td>164.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.385774e+09</td>
      <td>2020.000000</td>
      <td>3887.000000</td>
      <td>200.000000</td>
      <td>887.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 각 column 별로 결측치 확인
gamestats.isnull().sum()
```




    gameId            0
    subType           0
    gameLength        0
    teamId            0
    championId        0
    result            0
    position      33002
    dtype: int64




```python
# 'result' column에 있는 값 확인
gamestats['result'].unique()
```




    array(['WIN', 'LOSE', 'UNKNOWN'], dtype=object)



- result의 UNKNOWN은 "다시하기"이다.


```python
# 'UNKNOWN'인 case 확인
gamestats[gamestats['result'] == 'UNKNOWN']
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
      <th>gameId</th>
      <th>subType</th>
      <th>gameLength</th>
      <th>teamId</th>
      <th>championId</th>
      <th>result</th>
      <th>position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>300</th>
      <td>5385762506</td>
      <td>430</td>
      <td>196</td>
      <td>200</td>
      <td>134</td>
      <td>UNKNOWN</td>
      <td>M</td>
    </tr>
    <tr>
      <th>301</th>
      <td>5385762506</td>
      <td>430</td>
      <td>196</td>
      <td>200</td>
      <td>51</td>
      <td>UNKNOWN</td>
      <td>T</td>
    </tr>
    <tr>
      <th>302</th>
      <td>5385762506</td>
      <td>430</td>
      <td>196</td>
      <td>200</td>
      <td>23</td>
      <td>UNKNOWN</td>
      <td>A</td>
    </tr>
    <tr>
      <th>303</th>
      <td>5385762506</td>
      <td>430</td>
      <td>196</td>
      <td>100</td>
      <td>157</td>
      <td>UNKNOWN</td>
      <td>M</td>
    </tr>
    <tr>
      <th>304</th>
      <td>5385762506</td>
      <td>430</td>
      <td>196</td>
      <td>200</td>
      <td>12</td>
      <td>UNKNOWN</td>
      <td>S</td>
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
    </tr>
    <tr>
      <th>97396</th>
      <td>5385773777</td>
      <td>440</td>
      <td>194</td>
      <td>200</td>
      <td>51</td>
      <td>UNKNOWN</td>
      <td>A</td>
    </tr>
    <tr>
      <th>97397</th>
      <td>5385773777</td>
      <td>440</td>
      <td>194</td>
      <td>200</td>
      <td>238</td>
      <td>UNKNOWN</td>
      <td>M</td>
    </tr>
    <tr>
      <th>97398</th>
      <td>5385773777</td>
      <td>440</td>
      <td>194</td>
      <td>100</td>
      <td>16</td>
      <td>UNKNOWN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>97399</th>
      <td>5385773777</td>
      <td>440</td>
      <td>194</td>
      <td>200</td>
      <td>59</td>
      <td>UNKNOWN</td>
      <td>J</td>
    </tr>
    <tr>
      <th>97400</th>
      <td>5385773777</td>
      <td>440</td>
      <td>194</td>
      <td>100</td>
      <td>126</td>
      <td>UNKNOWN</td>
      <td>M</td>
    </tr>
  </tbody>
</table>
<p>670 rows × 7 columns</p>
</div>




```python
# 'UNKNOWN'인 case drop
gamestats = gamestats[gamestats['result'] != 'UNKNOWN']
```


```python
# 'subType' 확인
gamestats['subType'].value_counts()
```




    420     35380
    450     30850
    430     20030
    440     10928
    850       780
    830       710
    840       280
    2020      230
    2010      120
    2000       22
    Name: subType, dtype: int64




```python
# 솔랭(420), 칼바람(450)만 남기기
gamestats = gamestats[gamestats['subType'].isin([420,450])]
```

- 여러 종류의 게임 중 솔로 랭크와 칼바람만 남긴다.


```python
gamestats.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 66230 entries, 10 to 99991
    Data columns (total 7 columns):
     #   Column      Non-Null Count  Dtype 
    ---  ------      --------------  ----- 
     0   gameId      66230 non-null  int64 
     1   subType     66230 non-null  int64 
     2   gameLength  66230 non-null  int64 
     3   teamId      66230 non-null  int64 
     4   championId  66230 non-null  int64 
     5   result      66230 non-null  object
     6   position    35380 non-null  object
    dtypes: int64(5), object(2)
    memory usage: 4.0+ MB
    


```python
# 데이터 타입 categorical로 변환
gamestats['teamId'] = pd.Categorical(gamestats.teamId)
gamestats['subType'] = pd.Categorical(gamestats.subType)
gamestats['position'] = pd.Categorical(gamestats.position)
```

- 꼭 categorical로 변환하지 않아도 되지만 해당 컬럼들의 성질을 고려해 변환한다.


```python
# 솔랭 게임 중 포지션 분류 안 된 게임 확인
gamestats['position'][gamestats['subType'] == 420].isnull().sum()
```




    0



- 포지션 예측에 대해 만약 플레이어가 서폿인데 미드로 가서 던지고 하면 예측이 안된다고 한다.


- 현재 자료에선 그런 케이스는 없다.


```python
# 포지션 분류 안 된 게임 drop
gamestats = gamestats.drop(gamestats[(gamestats['subType']==420) & (gamestats['position'].isnull())].index)
```


```python
# 'WIN', 'LOSE' -> True, False로 다루기 용이하게 변환
gamestats['result'] = (gamestats['result'] == 'WIN')
```


```python
# 전처리된 DataFrame에서 row case 재확인
gamestats[['teamId','subType','result']].drop_duplicates().sort_values(["subType",'result','teamId'])
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
      <th>teamId</th>
      <th>subType</th>
      <th>result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>180</th>
      <td>100</td>
      <td>420</td>
      <td>False</td>
    </tr>
    <tr>
      <th>33</th>
      <td>200</td>
      <td>420</td>
      <td>False</td>
    </tr>
    <tr>
      <th>30</th>
      <td>100</td>
      <td>420</td>
      <td>True</td>
    </tr>
    <tr>
      <th>182</th>
      <td>200</td>
      <td>420</td>
      <td>True</td>
    </tr>
    <tr>
      <th>52</th>
      <td>100</td>
      <td>450</td>
      <td>False</td>
    </tr>
    <tr>
      <th>10</th>
      <td>200</td>
      <td>450</td>
      <td>False</td>
    </tr>
    <tr>
      <th>11</th>
      <td>100</td>
      <td>450</td>
      <td>True</td>
    </tr>
    <tr>
      <th>50</th>
      <td>200</td>
      <td>450</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
# subType, championId 별로 블루팀 승률, 레드팀 승률, 각 진영 게임 수 확인
def func_calc_WR(x):
        d = []
        d.append(sum(x['teamId']==100)),
        d.append(x['result'][x['teamId']==100].sum() / sum(x['teamId']==100)),
        d.append(sum(x['teamId']==200)),
        d.append(x['result'][x['teamId']==200].sum() / sum(x['teamId']==200)),
        
        return pd.Series(d, index=['BLUE_count', 'BLUE_WR', 'RED_count', 'RED_WR'])

summary = gamestats.groupby(['subType','championId'], as_index=False).apply(func_calc_WR)
```


```python
# Summary 확인
summary
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
      <th>subType</th>
      <th>championId</th>
      <th>BLUE_count</th>
      <th>BLUE_WR</th>
      <th>RED_count</th>
      <th>RED_WR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>420</td>
      <td>1</td>
      <td>34.0</td>
      <td>0.588235</td>
      <td>34.0</td>
      <td>0.588235</td>
    </tr>
    <tr>
      <th>1</th>
      <td>420</td>
      <td>2</td>
      <td>20.0</td>
      <td>0.700000</td>
      <td>28.0</td>
      <td>0.464286</td>
    </tr>
    <tr>
      <th>2</th>
      <td>420</td>
      <td>3</td>
      <td>91.0</td>
      <td>0.538462</td>
      <td>92.0</td>
      <td>0.489130</td>
    </tr>
    <tr>
      <th>3</th>
      <td>420</td>
      <td>4</td>
      <td>66.0</td>
      <td>0.424242</td>
      <td>69.0</td>
      <td>0.449275</td>
    </tr>
    <tr>
      <th>4</th>
      <td>420</td>
      <td>5</td>
      <td>154.0</td>
      <td>0.474026</td>
      <td>169.0</td>
      <td>0.449704</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>307</th>
      <td>450</td>
      <td>555</td>
      <td>208.0</td>
      <td>0.480769</td>
      <td>200.0</td>
      <td>0.455000</td>
    </tr>
    <tr>
      <th>308</th>
      <td>450</td>
      <td>777</td>
      <td>92.0</td>
      <td>0.500000</td>
      <td>89.0</td>
      <td>0.426966</td>
    </tr>
    <tr>
      <th>309</th>
      <td>450</td>
      <td>875</td>
      <td>101.0</td>
      <td>0.623762</td>
      <td>120.0</td>
      <td>0.450000</td>
    </tr>
    <tr>
      <th>310</th>
      <td>450</td>
      <td>876</td>
      <td>71.0</td>
      <td>0.591549</td>
      <td>68.0</td>
      <td>0.411765</td>
    </tr>
    <tr>
      <th>311</th>
      <td>450</td>
      <td>887</td>
      <td>42.0</td>
      <td>0.523810</td>
      <td>53.0</td>
      <td>0.377358</td>
    </tr>
  </tbody>
</table>
<p>312 rows × 6 columns</p>
</div>



- 게임 타입과 챔피언 별로 진영에 따른 판수와 승률을 추가하였다.


```python
# 유의수준 0.005에서 검증
summary['p_value'] = (2*norm.cdf(-abs(summary.RED_WR - summary.BLUE_WR)/
                                 np.sqrt(summary.BLUE_WR*(1-summary.BLUE_WR)/summary.BLUE_count+
                                         summary.RED_WR*(1-summary.RED_WR)/summary.RED_count)))
summary['unbalanced'] = summary['p_value'] < 0.005
```

- 각 진영별로 승률이 차이가 있는지 확인하기 위해 유의확률을 직접 구하였다.


- 기준을 0.005로 잡아 가설을 검증하였다.


```python
summary[summary['unbalanced']].sort_values('p_value')
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
      <th>subType</th>
      <th>championId</th>
      <th>BLUE_count</th>
      <th>BLUE_WR</th>
      <th>RED_count</th>
      <th>RED_WR</th>
      <th>p_value</th>
      <th>unbalanced</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>180</th>
      <td>450</td>
      <td>25</td>
      <td>192.0</td>
      <td>0.682292</td>
      <td>199.0</td>
      <td>0.462312</td>
      <td>0.000006</td>
      <td>True</td>
    </tr>
    <tr>
      <th>169</th>
      <td>450</td>
      <td>14</td>
      <td>43.0</td>
      <td>0.744186</td>
      <td>42.0</td>
      <td>0.380952</td>
      <td>0.000289</td>
      <td>True</td>
    </tr>
    <tr>
      <th>195</th>
      <td>450</td>
      <td>40</td>
      <td>135.0</td>
      <td>0.629630</td>
      <td>114.0</td>
      <td>0.412281</td>
      <td>0.000462</td>
      <td>True</td>
    </tr>
    <tr>
      <th>131</th>
      <td>420</td>
      <td>246</td>
      <td>49.0</td>
      <td>0.244898</td>
      <td>57.0</td>
      <td>0.526316</td>
      <td>0.001823</td>
      <td>True</td>
    </tr>
    <tr>
      <th>244</th>
      <td>450</td>
      <td>105</td>
      <td>101.0</td>
      <td>0.544554</td>
      <td>96.0</td>
      <td>0.343750</td>
      <td>0.003771</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 숫자로 된 각종 Key값에 대한 Value 가져오기
champion_constant = requests.get("http://ddragon.leagueoflegends.com/cdn/11.16.1/data/ko_KR/champion.json")
# Json 파일을 DataFrame으로 변환
champion_df = pd.DataFrame(champion_constant.json()['data']).T[['key','name']]
# 변수형 문자 -> 숫자
champion_df['key'] = pd.to_numeric(champion_df['key'])
```

- 1일차에서 배웠던 챔피언 정보를 불러온다.


```python
# 매칭되는 곳으로 내용 추가하여 새로운 DataFrame으로 저장
summary = pd.merge(summary,champion_df,how = 'left', left_on = 'championId', right_on = 'key')
# 중복되는 값 삭제
summary = summary.rename(columns={'name': 'Champion'}).drop(['championId','key'],axis = 1)
summary_View = summary[['subType','Champion','BLUE_WR','RED_WR','unbalanced','p_value']]
```


```python
summary_View[summary_View['unbalanced']].sort_values('p_value')
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
      <th>subType</th>
      <th>Champion</th>
      <th>BLUE_WR</th>
      <th>RED_WR</th>
      <th>unbalanced</th>
      <th>p_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>180</th>
      <td>450</td>
      <td>모르가나</td>
      <td>0.682292</td>
      <td>0.462312</td>
      <td>True</td>
      <td>0.000006</td>
    </tr>
    <tr>
      <th>169</th>
      <td>450</td>
      <td>사이온</td>
      <td>0.744186</td>
      <td>0.380952</td>
      <td>True</td>
      <td>0.000289</td>
    </tr>
    <tr>
      <th>195</th>
      <td>450</td>
      <td>잔나</td>
      <td>0.629630</td>
      <td>0.412281</td>
      <td>True</td>
      <td>0.000462</td>
    </tr>
    <tr>
      <th>131</th>
      <td>420</td>
      <td>키아나</td>
      <td>0.244898</td>
      <td>0.526316</td>
      <td>True</td>
      <td>0.001823</td>
    </tr>
    <tr>
      <th>244</th>
      <td>450</td>
      <td>피즈</td>
      <td>0.544554</td>
      <td>0.343750</td>
      <td>True</td>
      <td>0.003771</td>
    </tr>
  </tbody>
</table>
</div>



- 최종 자료를 보면 모르가나 사이온 등은 진영에 따라 승률 차이가 커보인다.


- 일반적으로 사람들이 블루팀이 유리하다고 생각하지만 여기서 유불리 유무는 진영에 따라 승률 차이가 있는지 검증이다.


- True라고 블루팀이 유리하다는 것이 아니며 앞서 유의확률 생성시 양쪽 검증 기준으로 만들었음을 기억하자.
