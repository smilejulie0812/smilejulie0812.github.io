---
title: "정보처리기사 Java 기출문제(1)"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - Java
sidebar:
  nav: "docs"
---
#### 2020년 1회
```java
class test1{
	public static void main(String[] args){
		int i;							// 'i'라는 정수형 변수 선언
		int a[] = {0, 1, 2, 3};			// 'a'라는 행렬을 선언 후, 그 값을 지정해줌

	for(i=0; i<4; i++){					// i가 0부터 3까지 하나씩 늘어나며 반복되는 for문을 사용하여,
		System.out.print(a[i] + " ");	// 행렬 a의 0열부터 3열까지의 값+스페이스를 줄줄이 출력한다.
		}
	}
}
```
>0 1 2 3

#### 2020년 1회
```java
class test2{
    public static void main(String[] args){
        int i = 3;		// 정수 i를 선언하고 값 3을 설정
        int k = 1;		// 정수 k를 선언하고 값 1을 설정
        
        switch(i){		// 입력변수 i를 갖는 switch문(어떤 변수의 값에 따라서 문장을 실행할 수 있도록 하는 제어문) 
            case 0:		// i=0일 경우 아무것도 하지 않음. 다만 i는 0이 아니므로 여기에 오지 않음
            case 1:		// i=1일 경우 아무것도 하지 않음. 다만 i는 1이 아니므로 여기에 오지 않음
            case 2:		// i=2일 경우 아무것도 하지 않음. 다만 i는 2가 아니므로 여기에 오지 않음
            case 3:		// i=3일 경우, k에 0값을 대입
                k=0;	// 이 단계에서 k=0이 됨. 또한 break가 없으므로 이대로 case 4로 넘어감
            case 4:		
                k+=3;	// 위에서 내려온 k에 3을 더한 값을 대입. 이 단계에서 k=0+3=3이 됨. 또한 break가 없으므로 이대로 case 5로 넘어감
            case 5:	
                k-=10;	// 위에서 내려온 k에 10을 뺀 값을 대입. 이 단계에서 k=3-10=-7이 됨. 또한 break가 없으므로 이대로 default로 넘어감
            default:	// 위의 값에 해당되지 않는 값이 주어질 경우, 바로 default로 넘어옴
                k--;	// 위에서 내려온 k에 1을 뺀 값을 대입. 이 단계에서 k=-7-1=-8이 됨.
        }
    
    System.out.print(k);// 위의 switch 문에서 최종적으로 입력된 k값인 -8을 출력한다.
    }
}
```
>-8

#### 2020년 2회
```java
class Parent{
    public void show(){							// 'void'는 return값 없이 쓰이는 메소드
        System.out.println("Parent");			// Parent 클래스에서는 'Parent' 문자열을 출력
    }
}
class Child extends Parent{						// Parent 클래스를 상속하는 Child 클래스 정의
    public void show(){ 
        System.out.println("Child");			// Child 클래스에서는 'Child' 문자열을 출력
    }
}
public class Main{								// Main 클래스를 public class로 정의
    public static void main(String[] args){		// 제일 먼저 실행되는 메소드 정의
        Parent pa = new Child();				// Parent 클래스의 메소드를 오버라이딩(Parent 클래스가 갖는 메소드를 Child 클래서에서 재정의)
        pa.show();								// Child로 오버라이딩된 Parent클래스의 show 값은 "Child"로 출력됨 
    }
}
```
>Child

* **오버라이딩**은 자식 클래스가 부모 클래스를 재정의하는 것은 가능하나, 부모 클래스가 자식 클래스를 재정의하여 사용하는 것은 불가능
{: .notice--info}

#### 2020년 2회
```java
class A{									// 클래스 A 정의
    private int a;							// 클래스 A에서만 사용하는 정수형 변수 a 선언
    public A(int a){						// 어디에서든 접근 가능한 public 메소드 A를 선언한 후, 
        this.a=a;							// 2번째 열에서 선언한 private 변수 a가 이 메소드의 인수 a와 같음을 명시함
    }
    public void display(){
        System.out.println("a="+a);			// 그리하여 위의 a값을 사용하여 문자열과 출력하는 메소드 display를 선언하였다.
    }
}

class B extends A{							// 클래스 A를 상속하는 클래스 B 정의
    public B(int a){						// 정수형 변수 a를 가진 메소드 B 선언
        super(a);							// 부모 클래스 A로부터 상속받은 변수 a를 사용하여,
        super.display();					// 또한 부모 클래스 A로부터 상속받은 메소드 display를 사용한다.
    }
}

public class Main{							// Main 클래스를 public class로 정의
    public static void main(String[] args){	// 제일 먼저 실행되는 메소드가 여기임을 명시
        B obj = new B(10);					// 10(정수)를 변수로 가진 클래스 B의 매소드를 실행시킨다. 여기서 클래스 B는 클래스 A를 그대로 상속받으므로, 클래스 A를 실행시키는 결과가 출력된다고 보면 됨
    }
}
```
>a=10

* **this**: 클래스 필드의 변수 자리에, 메소드의 매개변수값을 대입한다.
* **super**: 부모 클래스로부터 상속받은 변수, 메소드를 사용한다.
{: .notice--info}

#### 2020년 3회
```java
class test5{
    public static void main(String[] args){
        int i=0;
        int sum=0;
    
        while(i < 10){				// i가 10 미만일 때까지 아래 처리를 반복한다
            i++;					// i에 1을 추가한다
            if(i % 2 == 1)			// i를 2로 나누어 나머지값이 1일 경우,
                continue;			// 'i++'로 돌아가 처리를 반복한다.
            sum += i;				// i를 2로 나누어 나머지값이 1이 아닐 경우, 변수 sum에 i값을 더한다. 이 경우, 2+4+6+8+10 까지 반복될 것.
        }
        System.out.println(sum);	// i가 10 이상이 되어 while문을 벗어나왔을 때, 거기에 담긴 변수 sum의 값을 출력한다.
    }
}
```
>30

`while`문의 `continue`는 처리를 반복하는 의미, `break`는 처리를 중단하는 의미를 가짐.
{: .notice--info}
