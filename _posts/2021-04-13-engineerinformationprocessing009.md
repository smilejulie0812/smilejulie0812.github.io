---
title: "정보처리기사 2017년도 C언어 기출문제"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - C
sidebar:
  nav: "docs"
---
#### 2017년 1회
```c++
#include <stdio.h>

main(){
	int num[10];				// 정수 10개의 자리를 가진 배열 num을 정의
	int min=9999;
	int i;
	
	for(i=0; i<10; i++){
		scanf("%d", &num[i]);		// 배열 num에 정수를 입력
	}
	
	for(i=0; i<10; i++){
		if(min>num[i]){			// 배열 num 안의 정수가 min의 값보다 작으면,
			min=num[i];		// min의 값을 그 정수값으로 교환. 이를 전체 배열의 수로 반복하면 min값은 배열 내 정수 중 가장 작은 값이 남게 된다
		}
	}
	printf("가장 작은 값은 %d이다", min);	// 두 번째 for문을 통해 도출된 배열 num 내의 최소값을 문자열과 함께 출려
}
```
>입력된 10개의 값 중 최소값을 출력

#### 2017년 2회
```c++
#include <stdio.h>

int isprime(int number){
	int i;
	for(i=2; i<number; i++)
		if(number%i == 0)		// number를 i값으로 나누어서 그 나머지가 0인 경우
			return 0;		// 소수가 될 수 없으므로 0을 반환한다(기억하지 않음)
		return 1;			// 소수이므로 1을 반환한다(기억해둠)
}

int main(){
	int number=100, cnt=0, i;
	for(i=2; i<number; i++)
		cnt = cnt + isprime(i);					// 변수 cnt에 위의 isprime 클래스에서 1이 반환된 숫자의 개수를 더해준다
	printf("%d를 넘지 않는 소수는 %d개입니다.\n", number, cnt);		// 문자열과 함께 해당 숫자와, 그 숫자를 넘지 않는 소수의 개수를 출력
	return 0;
}
```
>100를 넘지 않는 소수는 25개입니다.

#### 2017년 3회
```c++
#include <stdio.h>

int res10(){
	return 4;				// 5) res10()=4
}

int res30(){
	return 30 + res10();			// 4) res30()=30+res10() -> 6) 30+4=34
}

int res200(){
	return 200 + res30();			// 3) res200=200+res30() -> 7) 200+34=234
}

int main(){					// 1) 가장 먼저 읽는 클래스는 main()
	int result;
	result=res200();			// 2) res200()값을 result에 대입시킨다 -> 8) res200()=234
	printf("%d\n", result);			// 9) 234를 출력
}
```
>234

#### 2017년 3회
```c++
#include <stdio.h>

int power(int data, int exp){
	int i, result=1;
	for(i=0; i<exp; i++)
		result = result * data;		// 초기값이 1로 설정되어있는 변수 result에 data를 exp회만큼 곱해준다(제곱값을 구하는 클래스)
	return result;
}

int main(){
	printf("%d\n", power(2,10));		// 2를 10회만큼 곱해주었을 때의 값 1024를 출력
	return 0;
}
```
>1024
