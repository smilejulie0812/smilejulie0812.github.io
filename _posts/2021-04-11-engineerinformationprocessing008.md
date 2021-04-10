---
title: "정보처리기사 2018년도 C언어 기출문제"
excerpt_separator: "<!--more-->"
categories:
  - Certificate
tags:
  - 정보처리기사
  - C
sidebar:
  nav: "docs"
---
#### 2018년 1회
```c++
#include <stdio.h>
#define MAX_STACK_SIZE 10

int stack[MAX_STACK_SIZE];			// 1) 대상 스택을 선언
int top = -1;					// 스택에서의 위치를 나타내는 변수 top을 선언, -1로 초기값을 설정함

void push(int item){				// 2) 수치를 대입할 수 있도록 item 변수를 갖는 push 클래스 생성
	if( top >= 9 ) {			// 위치가 9까지 도달해 있을 경우,
		printf("stack is full\n");	// stack is full 메시지를 출력하여 스택이 꽉 차 있음을 알림
	}
	stack[++top] = item;			// 그렇지 않을 경우(스택에 빈 공간이 존재할 경우), top에 1을 더해 위치를 설정한 후 item 값을 대입해줌
}

int pop(){					// 3) 스택에 들어있는 값을 내보내는 pop 클래스 생성
	if ( top == -1 ) {			// top의 값이 -1일 경우 아무 값도 들어있지 않다는 의미이므로,
		printf("stack is empty\n")	// stack is empty 메시지를 출력하여 스택이 비어 있음을 알림
	}
	return stack[top--];			// 그렇지 않을 경우(스택에 값이 존재할 경우), top에 1을 뺀 위치의 값을 반환한다
}

int inempty(){					// 4) 스택이 비어있을 경우를 가정하는 inempty 클래스 생성
	if ( top == -1 )
		return 1; else return 0;
}

int isfull(){					// 5) 스택이 꽉 차 있을 경우를 가정하는 isfull 클래스 생성
	if ( top >= 9 )
		return 1; else return 0;
}

int main(){					// 실행 클래스 main
	int e;
	push(20); push(30); push(40);		// 각각의 숫자 20, 30, 40을 스택에 대입하는 push 클래스 실행
	printf("stack's status\n");		// stack's status 메시지 출력
	
	while(!isempty()){			// isempty 클래스에 의해 1이 반환되지 않을 때까지(0일 때까지)
		e = pop();			// 각각의 pop 클래스에 의해 값이 반환된 e값을 가져와서
		printf("value = %d\n", e);	// 그 e값을 value = 라는 문자열과 함께 출력함
	}
}
```
>stack's status  
value = 20  
value = 30  
value = 40

<div class="notice--info" markdown="1">
**스택**의 기본적인 움직임을 이해하고 있다면 크게 어려운 문제는 아닌데, 개인적으로 ++N, N++ 등의 표현식에 좀 더 익숙해질 필요가 있을 것 같다.
</div>

#### 2018년 2회
```c++
#include <stdio.h>

main(){
	int i, a[5], cnt = 0;

	for(i=0; i<5; i++)
		scanf("%d", &a[i]);		// ★★★배열 a의 0~4번째 요소의 주소에 정수값을 입력하라는 의미

	for(i=0; i<5; i++){
		if(a[i] % 2 != 0)		// 위의 scanf에서 받은 정수가 홀수일 경우를 만들어줘야 하므로,
			cnt = cnt+1		// 정수를 2로 나눈 나머지가 0이 아니거나, 아니면 0보다 큰 경우를 만들어줘야 함(>0도 가능)
	}

	printf("홀수의 개수: %d개", cnt);
}
```
> 만약 1,2,3,4,5를 입력하면, 아래의 결과를 출력  
홀수의 개수: 3개
<div class="notice--info" markdown="1">
문제 자체는 쉽지만 아직도 C의 포인터 개념이 확실히 잡히지 않는다... 조만간 정리해서 올려둬야겠다
</div>

#### 2018년 2회
```c++
#include <stdio.h>

main(){
	int i,j;
	for(i=1; i<=5; i++){				// 문제가 1~5까지의 정수에 대해 약수를 구하는 문제이고
		printf("%d의 약수: ", i);			// ㄱ) 변수 i의 쓰임을 보니 약수를 구하는 대상 숫자를 뜻하는 것임

		for(j=1; j<=5; j++){			// 전체적인 흐름을 보면 변수 j는 i의 약수인지를 판별하여 출력하는 수치이므로
			if( i % j == 0 )		// i에 j를 나누어 나머지가 0인지(j가 i의 약수인지)를 판별하는 조건문을 만들어줘야 함
				printf("%d", j);	// ㄴ) 해당하는 j값 출력
		}
		printf("\n");				// ㄱ)과 ㄴ)을 출력하면 i값이 바뀌게 되므로 그 때마다 개행을 넣어준다
	}
	return 0;
}
```
>1의 약수: 1  
2의 약수: 12  
3의 약수: 13  
4의 약수: 124  
5의 약수: 15

#### 2018년 3회
```c++
#include <stdio.h>
#include <stdlib.h> 					// stdlib.h는 rand() 함수를 포함하고 있는 라이브러리
#include <time.h>					// time.h는 time() 함수를 포함하고 있는 라이브러리

main(){
	int hist[6] = { 0, };
	int n,i = 0;
	srand(time(NULL));				// ★★★1초 단위로 매번 다른 시드값을 생성해 rand() 함수를 호출함

	do{
		i++;
		n = rand() % 6 + 1;			// ★★★어떤 난수를 6으로 나누었을 때 나오는 나머지(0~5)에 1씩 더하는 것으로 주사위의 눈(1~6)을 랜덤으로 구현할 수 있음
		hist[n-1] += 1;				// ★★★주사위 눈 n(1~6)에 대응하는 히스토그램의 배열 요소는 n-1(0~6)이므로, n-1번째 배열의 히스토그램에 눈이 나온 횟수를 1씩 더해준다
	}while(i<100);					// 이걸 100번씩 반복해준다(i가 0부터 시작하므로 99번째까지 반복하는 do while문)

	for(i=0; i<6; i++)				// 위에서 구한 히스토그램을 출력
		printf("[%d] = $d\n", i+1, hist[i] );	//i+1은 주사위 눈(1~6)이고, hist[i]는 주사위 눈에 해당하는 히스토그램에 쌓인 각 눈이 나온 횟수를 의미한다
}
```
>
**rand()**: 랜덤 숫자를 대입하는 함수  
**srand(time(NULL))**: 1초 단위로 매번 다른 시드값을 생성하여 rand() 함수를 호출하는 함수  
**hist[]**: 히스토그램 그리는 함수
<div class="notice--info" markdown="1">

</div>

#### 2018년 3회
```c++
#include <stdio.h>
#include <stdlib.h> 						// stdlib.h는 rand() 함수를 포함하고 있는 라이브러리

struct NODE{							// 연결 리스트(노드들의 집합)의 노드 구조체를 정의
	int data;						// 데이터를 저장할 멤버를 정의
	struct NODE *Next;					// NODE 자료형 포인터 변수 Next를 변수로 선언(★자기 자신이 아닌 다른 노드의 메모리 주소를 저장)
}

struct NODE *head;						// NODE 자료형 포인터 변수 head를 변수로 선언

void Push(int data){						// 정수형 변수 data를 가진 클래스 Push를 정의(리스트에 숫자를 넣는 용도)
	struct NODE *end = malloc(sizeof(struct NODE));		// end라는 새 노드를 할당하는데,
	end -> Next = head -> Next;				// 노드 end의 Next에는 노드 head의 Next를 저장하는데, ㄱ)에 의하면 head의 Next는 Null이므로 end의 Next에는 Null이 지정된다
	end -> data = data;					// 노드 end의 데이터에는 데이터값을 저장하고,
	head -> Next = end;					// 노드 head의 Next는 end가 가지고 있는 값을 저장한다
}

int Pop(){
	int a;

	struct NODE *del = head -> Next;			// NODE 자료형 포인터 변수 del을 변수로 선언하고, head의 Next를 저장. 즉, 노드 del의 용도는 리스트 구조에서 제거할 노드의 시작 주소를 저장하는 것임
	head -> Next = del -> Next;				// 노드 head의 Next에 노드 del의 Next를 저장. 즉, head의 Next는 리스트 구조에서 항상 첫 번째 노드를 가리키게 됨
	a = del -> data;					// 위에서 선언한 변수 a에 del의 데이터를 저장
	free(del);						// 노드 del이 가리키고 있는 메모리 공간을 해제하고,
	return a;						// 노드 del에 저장되어 있던 데이터 a를 호출
}

main(){
	int r;

	head = malloc(sizeof(struct NODE));			// head라는 새 노드를 할당하는데,
	head -> Next = NULL;					// ㄱ) head의 Next값이 NULL임. ★즉, head 노드가 맨 첫 번째 노드임

	Push(10);
	Push(20);
	Push(30);

	r = Pop();
	printf("%d\n", r);
	r = Pop();
	printf("%d\n", r);
	r = Pop();
	printf("%d\n", r);
}
```
>30  
20  
10

<div class="notice--info" markdown="1">
**연결 리스트(Linked List)**:  
각 노드가 데이터와 포인터를 가지고 한 줄로 연결되어 있는 방식으로, 데이터를 저장하는 자료 구조.  
데이터를 담고 있는 노드들이 연결되어 있는데, 노드의 포인터가 다음이나 이전 노드와의 연결을 담당한다.  
</div>

정처기 공부한다고 뜻하지 않게 C언어로도 알고리즘을 접한다ㅋㅋㅋ 파이썬은 커녕 자바로도 알고리즘까진 안배웠었는데...  
그리고 사실 아직 코드가 완벽하게 이해되지 않은 듯. 알고리즘 공부할 때 더 확실히 익혀둬야겠다.
{: .notice--danger}
