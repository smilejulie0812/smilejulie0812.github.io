---
title: "정보처리기사 2018년도 Java 기출문제"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - Java
sidebar:
  nav: "docs"
---
#### 2018년 1회
```java
public class Test1{
	public static void main(String[] args){
		int E[] = { 95, 75, 85, 100, 50 };	// 정렬할 함수 E 선언
		int i = 0;
		int Temp = 0;
		
		do{					// 이 do는 아래 while(i<4)와 세트인 while문
			int j = i;
			do{				// 이 do는 아래 while(j<5)와 세트인 while문
				if( E[i] > E[j] ){	// E[0]값을 E[1]~E[4]의 값까지 비교해서 E[0]이 클 경우,
					Temp = E[i];
					E[i] = E[j];
					E[j] = Temp;	// E[0]값과 비교값을 교환한다.
				}
				j++;			// 내부 while문에서는 j값만 증가시킴. 즉, 내부 while문을 한 번 돌릴 때마다 E[i]의 값에 배열의 최소값이 위치하게 됨
			} while( j<5 );
			i++;				// 외부 while문에서는 i값이 증가되므로, E[0]~E[3]까지 순서대로 값이 정렬됨
		} while( i<4 );
		
		for(int a=0; a<5; a++){
			System.out.printf(E[a]+"\t");	// 위의 이중 while문으로부터 정렬된 배열 E의 값을 탭과 함께 출력
		}
		System.out.println();
	}
}
```
>50	75	85	95	100

<div class="notice--info" markdown="1">
`do~while`문의 사용법: do { 처리 } while( 조건문 )
</div>

#### 2018년 2회
```java
public class Problem{
	public static void main(String[] args){
		int[][] a = new int[3][5];
		for(int i=0; i<3; i++){					// 행렬 a의 행을 0부터 2까지 돌림
			for(int j=0; j<5; j++){				// 행렬 a의 열을 0부터 4까지 돌림
				a[i][j] = i+j;				// 행렬 a의 각 값은 '행의 수+열의 수' 로 한다
				System.out.printf("%d", a[i][j]);	// 행렬 a의 각 값을 출력
			}
			System.out.println();
		}
	}
}
```
>0 1 2 3 4  
1 2 3 4 5  
2 3 4 5 6

#### 2018년 3회
```java
public class Problem{
	public static void main(String[] args){
		int a, b, c, sum;
		a = b = 1;
		sum = a+b;

		for(int i=3; i<=5; i++){
			c = a+b;
			sum += c;
			a = b;
			b = c;
		}
		System.out.println(sum);
	}
}
```
>12

<div class="notice--info" markdown="1">
이런 문제같은 경우는 변수별로 표를 만들어서 확인하는 게 최고임
<html>
|:--------:|:--------:|:--------:|:--------:|  
| **i** | 3 | 4 | 5 |  
| **a** | 1 | 1 | 2 |  
| **b** | 1 | 2 | 3 |  
| **c** | 2 | 3 | 5 |  
| **sum** | 4 | 7 | 12 |  
| **a(next)** | 1 | 2 | 3 |  
| **b(next)** | 2 | 3 | 5 |  
</html>
</div>
