---
title: "[Python] 나도코딩 웹 스크래핑편 - (4) BeautifulSoup"
excerpt: "BeautifulSoup 패키지"
categories: 
  - 나도코딩
tags: Python, 나도코딩
# toc: TRUE
# toc_sticky: TRUE
use_math: TRUE
---

유튜브 [나도코딩 웹 스크래핑](https://www.youtube.com/watch?v=yQ20jZwDjTE&t=17499s) 무료 강의를 통해 학습한 내용을 정리해서 올리고 있습니다.

실습과정에서 필요에 따라 일부 강의 내용의 누락 및 추가, 수정사항이 있습니다.

퀴즈의 경우, 유튜브 풀이와 상이할 수 있습니다.

---


**BeautifulSoup**


```python
import requests
from bs4 import BeautifulSoup

# 네이버 웹툰
url = "https://comic.naver.com/webtoon/weekday.nhn"
res = requests.get(url)
res.raise_for_status()

# pip install lxml
soup = BeautifulSoup(res.text, "lxml") # soup 객체의 html 정보를 모두 담음
```

- BeautifulSoup은 웹 스크래핑을 위한 많은 기능이 있는 패키지이다.


- BeautifulSoup 객체를 통해서 많은 웹 정보를 확인 가능하다.


- 여기선 네이버 웹툰 홈페이지의 정보를 가져와보자.


```python
print(soup.title)
print(soup.title.get_text())
```

    <title>네이버 만화 &gt; 요일별  웹툰 &gt; 전체웹툰</title>
    네이버 만화 > 요일별  웹툰 > 전체웹툰
    

- html에서 title 정보 및 title의 text를 확인하였다.


```python
print(soup.a)
```

    <a href="#menu" onclick="document.getElementById('menu').tabIndex=-1;document.getElementById('menu').focus();return false;"><span>메인 메뉴로 바로가기</span></a>
    

- soup 객체에서 처음 발견되는 a 태그 정보를 출력한다.


```python
print(soup.a.attrs)
```

    {'href': '#menu', 'onclick': "document.getElementById('menu').tabIndex=-1;document.getElementById('menu').focus();return false;"}
    

 - a 태그의 속성 정보를 출력한다.


```python
print(soup.a["href"]) # a element 의 href 정보
```

    #menu
    

 - a 태그의 href 속성 정보를 출력한다.


```python
soup.find("a", attrs={"class":"Nbtn_upload"})
# soup.find(attrs={"class":"Nbtn_upload"})
```




    <a class="Nbtn_upload" href="/mypage/myActivity.nhn" onclick="nclk_v2(event,'olk.upload');">웹툰 올리기</a>



 - class가 Nbtn_upload인 a 태그를 찾는다.


```python
# li tag 밑의 tag 정보 
rank1 = soup.find("li", attrs={"class":"rank01"})
print(rank1.a)
print(rank1.a.get_text())
```

    <a href="/webtoon/detail.nhn?titleId=747269&amp;no=55" onclick="nclk_v2(event,'rnk*p.cont','747269','1')" title="전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)">전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)</a>
    전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)
    

 - class가 rank01인 li 태그를 찾은 후 li 태그 밑의 첫 번째 a 태그 정보, text를 출력하였다.


```python
# 형제 찾기
rank2 = rank1.next_sibling.next_sibling
rank3 = rank2.next_sibling.next_sibling
print(rank3.a.get_text())

rank2 = rank3.previous_sibling.previous_sibling
print(rank2.a.get_text())

# 형제 찾기2
rank2 = rank1.find_next_sibling("li")
print(rank2.a.get_text())

rank2 = rank3.find_previous_sibling("li")
print(rank2.a.get_text())
```

    복학왕-347화 다들 먹고 산다 1화
    급식아빠-21화 선 넘네?
    급식아빠-21화 선 넘네?
    급식아빠-21화 선 넘네?
    

 - class가 rank01인 li 태그는 class가 rank02, rank03, .. 등인 형제 li 태그가 존재한다.
 
 
 - `next_sibling` 속성을 이용해서 다음 형제 태그를 찾을 수 있고 `previous_sibling` 속성으로 이전 형제 태그를 찾을 수 있다.
 
 
 - 여기서 `next_sibling` 등을 2번 사용한 이유는 바로 다음 형제가 클래스가 rank02인 li 태그가 아니어서이다.
 
 
 - `find_next_sibling()`, `find_previous_sibling()` 함수로 태그를 지정해서 다음, 이전 지정 태그를 찾을 수 있다.


```python
# 부모 찾기
print(rank1.parent)
```

    <ol class="asideBoxRank" id="realTimeRankFavorite">
    <li class="rank01">
    <a href="/webtoon/detail.nhn?titleId=747269&amp;no=55" onclick="nclk_v2(event,'rnk*p.cont','747269','1')" title="전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)">전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>
    <li class="rank02">
    <a href="/webtoon/detail.nhn?titleId=758662&amp;no=21" onclick="nclk_v2(event,'rnk*p.cont','758662','2')" title="급식아빠-21화 선 넘네?">급식아빠-21화 선 넘네?</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>
    <li class="rank03">
    <a href="/webtoon/detail.nhn?titleId=626907&amp;no=349" onclick="nclk_v2(event,'rnk*p.cont','626907','3')" title="복학왕-347화 다들 먹고 산다 1화">복학왕-347화 다들 먹고 산다 1화</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>
    <li class="rank04">
    <a href="/webtoon/detail.nhn?titleId=670143&amp;no=255" onclick="nclk_v2(event,'rnk*p.cont','670143','4')" title="헬퍼 2 : 킬베로스-254화. ■">헬퍼 2 : 킬베로스-254화. ■</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>
    <li class="rank05">
    <a href="/webtoon/detail.nhn?titleId=738694&amp;no=59" onclick="nclk_v2(event,'rnk*p.cont','738694','5')" title="튜토리얼 탑의 고인물-59화">튜토리얼 탑의 고인물-59화</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>
    <li class="rank06">
    <a href="/webtoon/detail.nhn?titleId=749639&amp;no=47" onclick="nclk_v2(event,'rnk*p.cont','749639','6')" title="남주의 첫날밤을 가져버렸다-47화">남주의 첫날밤을 가져버렸다-47화</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>
    <li class="rank07">
    <a href="/webtoon/detail.nhn?titleId=750184&amp;no=45" onclick="nclk_v2(event,'rnk*p.cont','750184','7')" title="나쁜사람-45화">나쁜사람-45화</a>
    <span class="rankBox">
    <img alt="순위상승" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_up.gif" title="순위상승" width="7"/>1
    						
    						
    					
    				</span>
    </li>
    <li class="rank08">
    <a href="/webtoon/detail.nhn?titleId=738143&amp;no=75" onclick="nclk_v2(event,'rnk*p.cont','738143','8')" title="여주실격!-72화 적을 더 가까이 두라 (3)">여주실격!-72화 적을 더 가까이 두라 (3)</a>
    <span class="rankBox">
    <img alt="순위하락" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_down.gif" title="순위하락" width="7"/>1
    						
    						
    						
    					
    				</span>
    </li>
    <li class="rank09">
    <a href="/webtoon/detail.nhn?titleId=710747&amp;no=130" onclick="nclk_v2(event,'rnk*p.cont','710747','9')" title="세상은 돈과 권력-시즌2 23화">세상은 돈과 권력-시즌2 23화</a>
    <span class="rankBox">
    <img alt="순위상승" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_up.gif" title="순위상승" width="7"/>1
    						
    						
    					
    				</span>
    </li>
    <li class="rank10">
    <a href="/webtoon/detail.nhn?titleId=728015&amp;no=220" onclick="nclk_v2(event,'rnk*p.cont','728015','10')" title="모죠의 일지-220화. 재봉틀 발굴한 만화 (1)">모죠의 일지-220화. 재봉틀 발굴한 만화 (1)</a>
    <span class="rankBox">
    <img alt="순위하락" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_down.gif" title="순위하락" width="7"/>1
    						
    						
    						
    					
    				</span>
    </li>
    </ol>
    

- `parent` 속성을 이용해서 li 태그의 부모 태그인 ol 태그 정보를 가져올 수 있다.


```python
# 한번에 형제 찾기
print(rank1.find_next_siblings("li"))
```

    [<li class="rank02">
    <a href="/webtoon/detail.nhn?titleId=758662&amp;no=21" onclick="nclk_v2(event,'rnk*p.cont','758662','2')" title="급식아빠-21화 선 넘네?">급식아빠-21화 선 넘네?</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>, <li class="rank03">
    <a href="/webtoon/detail.nhn?titleId=626907&amp;no=349" onclick="nclk_v2(event,'rnk*p.cont','626907','3')" title="복학왕-347화 다들 먹고 산다 1화">복학왕-347화 다들 먹고 산다 1화</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>, <li class="rank04">
    <a href="/webtoon/detail.nhn?titleId=670143&amp;no=255" onclick="nclk_v2(event,'rnk*p.cont','670143','4')" title="헬퍼 2 : 킬베로스-254화. ■">헬퍼 2 : 킬베로스-254화. ■</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>, <li class="rank05">
    <a href="/webtoon/detail.nhn?titleId=738694&amp;no=59" onclick="nclk_v2(event,'rnk*p.cont','738694','5')" title="튜토리얼 탑의 고인물-59화">튜토리얼 탑의 고인물-59화</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>, <li class="rank06">
    <a href="/webtoon/detail.nhn?titleId=749639&amp;no=47" onclick="nclk_v2(event,'rnk*p.cont','749639','6')" title="남주의 첫날밤을 가져버렸다-47화">남주의 첫날밤을 가져버렸다-47화</a>
    <span class="rankBox">
    <img alt="변동없음" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_no.gif" title="변동없음" width="7"/> 0
    						
    					
    				</span>
    </li>, <li class="rank07">
    <a href="/webtoon/detail.nhn?titleId=750184&amp;no=45" onclick="nclk_v2(event,'rnk*p.cont','750184','7')" title="나쁜사람-45화">나쁜사람-45화</a>
    <span class="rankBox">
    <img alt="순위상승" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_up.gif" title="순위상승" width="7"/>1
    						
    						
    					
    				</span>
    </li>, <li class="rank08">
    <a href="/webtoon/detail.nhn?titleId=738143&amp;no=75" onclick="nclk_v2(event,'rnk*p.cont','738143','8')" title="여주실격!-72화 적을 더 가까이 두라 (3)">여주실격!-72화 적을 더 가까이 두라 (3)</a>
    <span class="rankBox">
    <img alt="순위하락" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_down.gif" title="순위하락" width="7"/>1
    						
    						
    						
    					
    				</span>
    </li>, <li class="rank09">
    <a href="/webtoon/detail.nhn?titleId=710747&amp;no=130" onclick="nclk_v2(event,'rnk*p.cont','710747','9')" title="세상은 돈과 권력-시즌2 23화">세상은 돈과 권력-시즌2 23화</a>
    <span class="rankBox">
    <img alt="순위상승" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_up.gif" title="순위상승" width="7"/>1
    						
    						
    					
    				</span>
    </li>, <li class="rank10">
    <a href="/webtoon/detail.nhn?titleId=728015&amp;no=220" onclick="nclk_v2(event,'rnk*p.cont','728015','10')" title="모죠의 일지-220화. 재봉틀 발굴한 만화 (1)">모죠의 일지-220화. 재봉틀 발굴한 만화 (1)</a>
    <span class="rankBox">
    <img alt="순위하락" height="10" src="https://ssl.pstatic.net/static/comic/images/migration/common/arrow_down.gif" title="순위하락" width="7"/>1
    						
    						
    						
    					
    				</span>
    </li>]
    

- `find_next_siblings()` 태그를 지정해서 다음으로 나오는 같은 태그를 모두 찾을 수 있다.


```python
# text를 이용해서 찾기 
webtoon = soup.find("a", text="전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)")
print(webtoon)
```

    <a href="/webtoon/detail.nhn?titleId=747269&amp;no=55" onclick="nclk_v2(event,'rnk*p.cont','747269','1')" title="전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)">전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)</a>
    

- 앞서 class를 지정하고 태그를 찾은 것 처럼 text를 지정해서 태그를 찾을 수도 있다.


- 미리 확인한 text를 적어주었고 시간별로 네이버 웹툰 상위 순위가 바뀌기 떄문에 지금 text가 없을 수도 있다.


```python
# 인기 급상승 만화 확인
# 형제 찾기로 rank02부터 class 확인 후 for문 이용해서 text 출력
rank = rank1.find_next_siblings("li")

print("1위:", rank1.a.get_text())
for row, i in enumerate(rank):
    print(f"{row + 2}위: {i.a.get_text()}")
```

    1위: 전지적 독자 시점-054. Ep.12 1인칭 주인공 시점 (2)
    2위: 급식아빠-21화 선 넘네?
    3위: 복학왕-347화 다들 먹고 산다 1화
    4위: 헬퍼 2 : 킬베로스-254화. ■
    5위: 튜토리얼 탑의 고인물-59화
    6위: 남주의 첫날밤을 가져버렸다-47화
    7위: 나쁜사람-45화
    8위: 여주실격!-72화 적을 더 가까이 두라 (3)
    9위: 세상은 돈과 권력-시즌2 23화
    10위: 모죠의 일지-220화. 재봉틀 발굴한 만화 (1)
    

- `find_next_siblings()`로 네이버 웹툰 상위 순위 형제 정보를 모두 저장 후 text를 출력하였다.