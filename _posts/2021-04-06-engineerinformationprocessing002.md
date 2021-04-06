---
title: "정보처리기사 2020년 기출문제 (Java 2)"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - Java
sidebar:
  nav: "docs"
---
#### 2020년 3회
```java
abstract class Vehicle{
	private String name;
	abstract public String getName(String val);
	public String getName(){
		return "Vehicle name:" + name;	// 6) 5)로부터 전달받은대로, 해당 문자열과 4)에서 저장한 name 변수의 값(Spark)를 반환받는다
	}
	
	public void setName(String val){
		name = val;	// 4) 2)로부터 3)을 통해 전달받은 변수 Spark를 name이라는 변수에 저장한다
	}
}

class Car extends Vehicle{
	public Car(String val){
		setName(val);	// 3) Car 메소드는 추상 클래스 Vehicle의 setName 메소드를 사용한다
	}
	
	public String getName(byte val[]){
		return "Car name:" + val;
	}
}

public class main{	// 1) main 함수에서부터 시작
	public static void main(String[] args){
		Vehicle obj = new Car("Spark");	// 2) Spark라는 매개변수를 가진 Car라는 생성자를 생성한다(이 때 obj는 추상 클래스 Vehicle의 변수이다)
		System.out.print(obj.getName());	// 5) 한편, getName 매소드의 매개변수가 비어있으므로, 클래스 Car가 아닌 클래스 Vehicle의 매소드를 사용해야 한다 -> 7) 6) 으로부터 반환받은 Vehicle name:Spark를 출력한다
	}
}
```
>Vehicle name:Spark

* `public Car(String val){` 이 부분에서 오류가 나는데... 그 원인은 좀 더 연구해보고 추가하는 것으로 한다.
{: .notice--info}

#### 2020년 4회
```java
class Exam{
    public static void main (String[] args){
        int []a = new int[8];	// 8개의 자리가 있는 새 배열을 생성
        int i = 0;	// i의 초기값 0
        int n = 10;	// n의 초기값 10(2진수로 변환할 10진수 숫자를 여기에 대입)
        
        while(i<8){	// i를 0부터 7까지 반복한다(a[0]~a[7]까지 숫자를 채우기 위함)
            a[i++] = n%2;	// 숫자 10의 나머지를 구한다(그 나머지가 이진수가 되므로)
            n /= 2;	// 숫자를 2로 나누어준다
        }
        
        for(i=7; i>=0; i--){	// 위의 for문에서 구한 배열 a의 역순이 2진수가 되므로, 뒤에서부터 거꾸로 for문을 돌려준다
            System.out.print(a[i]);	// 거꾸로 for문을 돌리며 배열의 숫자를 출력한다
        }
    }
}
```
>00001010

#### 2020년 4회
```java
public class Exam{
    public static void main(String[] args){
        int[][] a = new int[3][5];  // 3행 5열의 새로운 배열 a를 선언
        for(int i=0; i<3; i++){ // i(행수)를 0부터 2까지 증가시키며 for문 실행
            for(int j=0; j<5; j++){ // j(열수)를 0부터 4까지 증가시키며 for문 실행 
                a[i][j] = j*3 + (i+1);  // 이중 for문 안에서 행(i)은 1씩 증가, 열(j)는 3씩 증가하는 규칙을 지정해줌 
                System.out.print(a[i][j] + " ");    // 결과를 출력
            }
            System.out.println();   // 결과를 한 행씩 출력
        }
    }
}
```
>1 4 7 10 13
>2 5 8 11 14
>3 6 9 12 15

#### 2020년 2회
```java
class Parent{
    public int compute(int num){
        if(num <= 1) return num;
        return compute(num-1) + compute(num-2);
    }
}

class Child extends Parent{
    public int compute(int num){
        if(num <= 1) return num;    // 3) 4<=1 이 아니므로,
        return compute(num-1) + compute(num-3); // 4) compute(3)+compute(1)을 출력해야 하는데,(여기서 compute(1)은 1)
            // compute(3)은 3<=1 이 아니므로, compute(2)+compute(0)을 출력해야 함(여기서 compute(0)=0)
            // compute(2)는 2<=1 이 아니므로, compute(1)+compute(-1)을 출력해야 함(여기서 compute(1)=1, compute(-1)=-1)
    }
}

class Exam{
    public static void main(String[] args){
        Parent obj = new Child();   // 1) obj는 Child 인스턴스를 가진다.
        System.out.print(obj.compute(4));   // 2) obj.compute(4)는 Child 클래스의 compute 메서드를 실행시키라는 의미
            // 5) 4)에서 모인 값을 계산하면, compute(4) = compute(3)+compute(1) = 0+1 = 1 즉, 1이 출력되어야 함.
    }
}
```
>1