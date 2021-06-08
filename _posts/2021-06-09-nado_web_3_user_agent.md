---
title: "[Python] 나도코딩 웹 스크래핑편 - (3) User-Agent"
excerpt: "정규식 패키지 re"
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


**User-Agent**

```python
import requests

headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.104 Safari/537.36"}
url = "https://romg2.github.io"

res = requests.get(url, headers = headers)
res.raise_for_status()

with open("./romg2.html", "w", encoding="utf8") as f:
    f.write(res.text)
```

- user agent string이라고 구글에 검색하면 나의 User-Agent를 확인 가능하다.


- User-Agent는 내가 어떤 OS를 쓰는지 버전은 뭔지 웹 브라우저의 정보는 무엇인지 등의 정보를 가지고 있다.


- 웹 스크래핑시에 간혹 스크래핑이 안되는 경우가 있는데 이는 스크립트가 로봇으로 판단하여 거부하는 경우이다.


- 따라서 웹 스크래핑할 때 User-Agent 정보를 입력해주어 웹 정보를 가져온다.


- User-Agent는 `request` 패키지의 `get()`함수에 headers에 입력해준다.
