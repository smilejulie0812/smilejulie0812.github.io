---
title: "정보처리기사 2020년도 Python 기출문제"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - Python
sidebar:
  nav: "docs"
---
#### 2020년 2회
```python
a={'일본', '중국', '한국'}
a.add('베트남')						# 새로운 값 추가
a.add('중국')						# 이미 존재하는 값이므로 무시함
a.remove('일본')						# 기존 값 삭제
a.update({'홍콩', '한국', '태국'})		# 새로운 세트를 추가로 붙임. 이미 존재하는 값은 무시함
print(a)
```
>{'중국', '한국','베트남', '홍콩', '태국'}


#### 2020년 3회
```pyyhon
lol=[[1, 2, 3], [4, 5], [6, 7, 8, 9]]
print(lol[0])						# lol[0]은 맨 첫 번째 array이므로 [1, 2, 3]을 출력
print(lol[2][1])					# lol[2][1]은 세 번째 array의 두 번째 요소를 의미하므로 해당되는 숫자 7을 출력

for sub in lol:						# array인 lol를 새로 정의한 변수 sub의 순서대로 반복하고(여기서 sub은 행마다 주어진 array를 의미),
    for item in sub:				# sub 역시 새로 정의한 변수 item의 순서대로 반복한다(여기서 item은 행렬마다 주어진 숫자를 의미)
        print(item, end='')			# item(행렬)을 띄어쓰기 없이 순서대로 출력한다
    print()							# 위에서 출력한 결과를 sub별로 출력한다
```
>[1, 2, 3]  
7  
123  
45  
6789