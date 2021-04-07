---
title: "정보처리기사 2019년도 Java 기출문제"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - Java
sidebar:
  nav: "docs"
---
#### 2019년 1회
```java
class SuperObject{
	public void print(){
		draw();
	}
	
	public void draw(){				// 5) 4)에서 설명한 바와 같이, SuperObject 클래스의 draw 메소드를 실행
		draw();					// ★6) 이 draw 메소드는 1)에서 클래스 형 변환을 수행하였고, draw 메소드 자체가 자식 클래스인 SubObject 클래스에 의해 오버라이딩 되었으므로, SubObject의 draw 메소드를 수행한다
		System.out.println("Super Object");	// 8) 6)과 7)에 거쳐 draw 메소드를 실행했으므로, 해당 문자열을 출력한다
	}
}

class SubObject extends SuperObject{
	public void paint(){				// 3) 2)에서 설명한 바와 같이, SubObject 클래스의 paint 메소드를 실행
		super.draw();				// 4) 여기에서 부모 클래스인 SuperObject 클래스의 draw 메소드에 접근하게 됨
	}
	
	public void draw(){
		System.out.println("Sub Object");	// 7) 6)에서 설명한 바와 같이, 자식 클래스인 SubObject의 메소드 draw를 실행, 문자열을 출력한다
	}
}

public class Test{
	public static void main(String[] args){
		SuperObject a = new SubObject();	// ★1) 자식 클래스인 SubObject를 SuperObject 변수 타입으로 지정하여 새롭게 선언함
		a.paint();				// 2) a라는 변수로 paint 메소드를 호출하는데, a는 SubObject 클래스로 생성되었으므로, SubObject 클래스의 paint 메소드를 실행함
	}
}
```
>Sub Object  
Super Object

**{부모클래스 변수타입} {변수명} = new {자식클래스}();**의 쓰임이 아직 완벽하게 이해되지 않은 듯. 좀 더 공부해보자
{: .notice--info}

#### 2019년 1회
```java
public class Test{
	public static void main(String[] args){
		int i, sum = 0;
		for(i=1; i<=110; i++){  		// i를 1에서 109까지 1씩 더해가면서 아래 작업을 반복한다
			if(i%4 == 0)			// i를 4로 나누었을 때 나머지가 0인 경우에 한하여(4의 배수인 경우),
				sum = sum + 1;		// 변수 sum에 1씩 더해간다. 즉, 4의 배수인 i의 갯수를 찾는다
		}
		System.out.printf("%d", sum);		// for문이 끝나 4의 배수인 i의 갯수 sum값을 출력한다
	}
}
```
>27

#### 2019년 2회
```java
public class Test{
	public static void main(String[] args){
		int numAry[] = new int[5];		// 5개의 정수를 담는 배열인 numAry를 정의
		int result = 0;				// 정수 result에 0을 설정
		
		for(int i=0; i<5; i++)			// 정수 i가 0~4일 때까지 1씩 증가시키며, 아래를 반복한다
			numAry[i] = i+1;		// numAry에 정수를 담는다. i[0]=1, i[1]=2, ... i[4]=5
		
		for(int i:numAry)			// 정수 i가 배열 numAry가 끝날 때까지 아래를 반복한다
			result += i;			// result에 i값을 더해나간다
			
		System.out.printf("%d", result);	// 2개의 for문에 의해 반환한 result값을 출력한다
	}
}
```
>15


<div class="notice--info" markdown="1">
java에서 for문 작성할 때 {} 필요 없나보다... 그래도 습관을 좋게 들이기 위해 개인적으로는 붙여써야지  
**i:numAry** 의 쓰임을 새로 알았다. 지정한 배열의 모든 수치를 for문으로 돌릴 때 반드시 기억해서 사용해야겠다
</div>


#### 2019년 2회
```java
public class Test{
	public static int[] arr(int[] a){		// 정수로 이루어진 배열 a를 선언
		int i, j, sw, temp, n=5;		// 정수형 변수들을 선언
		if(a[0] == 0 || a[0] < 1)		// 만약 a[0]이 0이거나 1보다 작으면,
			return a;			// 배열 a를 반환한다(배열이 이상할 경우를 대비하여 초기설정)
			
		for(i=0; i<n-1; i++){			// 변수 i를 0부터 3까지 1씩 증가시키며 for문으로 돌려줌
			sw = i;				// 변수 sw에 i값(0부터 3까지)을 대입하고,
			for(j=i+1; j<n; j++){		// 변수 j를 1부터 4까지 1씩 증가시키며 for문으로 돌려줌(이중 for문)
				if(a[j] > a[sw])	// 만약, a[1~4]이 a[0]보다 클 경우를 찾는데,
					sw=j;		// if문이 true인 경우, 변수 sw값에 j값을 대입한다. false인 경우, j를 1씩 늘려 다시 if문 실행
			}				// ★여기서 sw=j를 시행하는 이유는, a[sw]값을 해당 배열에서 가장 큰 숫자를 찾기 위함임
			temp = a[i];			// 안쪽의 for문이 일단 끝나면(i=0이었을 때), 임시 변수 temp에 a[0]값을 넣어두고,
			a[i] = a[sw];			// a[0] 자리엔 a[4]값(i=0이었을 때 위의 for문을 돌린 결과, 가장 큰 값이 a[4]값이었음)을 대입한 후, 
			a[sw] = temp;			// a[4] 자리엔 아까 temp에 넣어둔 a[0]값을 대입한다. 즉, a[0]값과 a[4]값을 바꿔주는 것.
		}					// 이 과정을 i=1, 2, 3이었을 때 반복한다(이 이중 for문을 통해, 배열은 큰 수부터 차례로 정렬되게 됨)
		return a;				// 이중 for문이 완료되면 배열 a를 반환해준다
	}
	
	public static void main(String[] args){
		int i;
		int n[]={4,3,5,2,10};			// 위의 클래스를 실행시킬 함수 n을 지정
		arr(n);					// 위 클래스의 arr 매소드를 실행시킨다
		for(i=0; i<5; i++)			// 메소드 실행 결과를, i를 0~4까지 1씩 증가시키며,
			System.out.println(n[i]);	// 배열 n의 값을 순서대로 출력한다
	}
}
```
>10  
5  
4  
3  
2

이런 류의 정렬 문제들은 변수들의 움직임을 잘 추적해 가면서 규칙을 이해햐는 것이 무엇보다 중요하다. 표로 흐름도를 그리는 것을 귀찮아하지 말자!
{: .notice--info}
