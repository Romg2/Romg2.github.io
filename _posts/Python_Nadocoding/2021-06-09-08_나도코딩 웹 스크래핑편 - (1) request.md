---
title: "[Python] 나도코딩 웹 스크래핑편 - (1) request"
excerpt: "request 패키지"
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


**웹 정보 가져오기**


```python
import requests

# 구글 정보를 저장
res = requests.get("http://google.com")
print("응답코드: ", res.status_code)
```

    응답코드:  200
    

- `requests`는 웹 정보를 가져올 수 있는 패키지로 `get()`함수를 이용해서 웹 정보를 저장한다.


- 정상적으로 가져왔는데 `status_code` 속성을 통해 확인 가능하며 200이면 정상이다.

**정상적으로 불러왔는지 확인**


```python
if res.status_code == requests.codes.ok:
   print("정상입니다.")
else:
   print("문제가 생겼습니다. [에러코드: ", res.status_code, "]")
```

    정상입니다.
    

- 정상적으로 불러왔는지 확인하는 방법으로 requests.codes.ok는 200이다.


```python
res.raise_for_status()
```

- `raise_for_status()`함수는 정상적으로 불러오지 못하면 종료하는 기능으로 이 방법으로 사용하는 것이 간편하다.

**html 텍스트 확인**


```python
print(len(res.text))
# print(res.text)
```

    14093
    

- `get()`함수로 저장한 웹 정보에서 `text` 속성을 이용하면 html 코드를 확인 가능하다.

**html파일 저장**


```python
with open("./mygoogle.html", "w", encoding="utf8") as f:
    f.write(res.text)
```
