---
title: "정보처리기사 2017년도 Java 기출문제"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - Java
sidebar:
  nav: "docs"
---
#### 2017년 1회
```java
public class Test001{
	public static void main(String[] args){
		int[] a = {3, 4, 10, 2, 5};
		int temp;
		
		for(int i=0; i<=3; i++){
			for(int j=i+1; j<=4; j++){
				if(a[i]<a[j]){		// 앞에 있는 숫자가 뒤에 있는 숫자보다 작을 경우
					temp=a[i];
					a[i]=a[j];
					a[j]=temp;		// 두 숫자의 위치를 변경, 즉 큰 숫자를 앞으로 오게 함
				}
			}
		}
		for(int i=0; i<5; i++)
			System.out.println(a[i]);
	}
}
```
>10  
5  
4  
3  
2

<div class="notice--info" markdown="1">

</div>

#### 2017년 2회
```java
public class Test001{
	public static void main(String[] args){
		int a=0; sum=0;
		while(a<10){
			a++;			// a=0에서부터 시작하여 일단 1을 더해주고 아래 if문을 실행. 즉 a=1
			if(a%2 == 1)	// a를 2로 나눈 나머지값이 1일 경우(홀수일 경우)
				continue;	// 액션 취하지 않고 while문을 돌리기
			sum += a;		// a를 2로 나눈 나머지값이 1이 아닐경우(0일경우, 짝수일경우), sum에 a를 더함
		}
		system.out.println(sum);	// 0에서부터 10까지의 짝수만이 모인 값의 합인 sum값 30을 출력 
	}
}
```
>30

#### 2017년 3회
```java
public class Test002{
	public static void main(String[] args){
		int a[] = {10, 30, 50, 70, 90};
		int i, max, min;
		max = a[0];				// 배열의 0번째 숫자를 초기 최대값으로 설정
		min = a[0];				// 배열의 0번째 숫자를 초기 최소값으로 설정
		for(i=0; i<5; i++){
			if( a[i] > max )	// i번째 숫자가 최대값보다 클 경우,
				max = a[i];		// 최대값을 i번째 숫자로 설정
			if( a[i] < min )	// i번째 숫자가 최소값보다 작을 경우,
				min = a[i];		// 최소값을 i번째 숫자로 설정
		}
		System.out.printf("%d\n", max);		// for문으로 결정된 최대값을 출력
		System.out.printf("%d/n", min);		// for문으로 결정된 최소값을 출력
	}
}
```
>90  
10
