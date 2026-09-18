---
title: "[자료구조와 알고리즘 with 파이썬] Chapter 01 스택"
date: 2026-09-18 00:00:00 +0900
categories: [Study, Data Structure]
tags: [data-structure, stack, python, recursion]
description: 스택의 개념과 배열 구현, 괄호 검사, 파이썬 활용, 시스템 스택과 재귀 정리
math: true
---

## 1. 스택이란?

스택(stack)은 **후입선출(LIFO: Last-in First-Out)** 자료구조이다. ex) 웹 브라우저의 이전 페이지로 이동

![스택의 구조](/assets/img/posts/stack-chapter1/stack-structure.png)

### 추상 자료형

대부분 프로그래밍 언어에서는 별다른 처리가 필요 없이 바로 사용할 수 있는 기본 자료형(int, float, str 등)을 제공한다. 

그러나 문제 해결을 위한 프로그램을 작성할 때 기본 자료형만으로는 불편하거나 비효율적일 수 있기 때문에 사용자가 새로운 자료형을 정의해 사용할 수 있도록 지원한다.

**추상화**를 통해 핵심 특징만 골라 단순화하고, 그 결과 만들어진 자료형이 **추상 자료형(ADT: Abstract Data Type)**이다.

> 💡 추상 자료형: 어떤 자료를 다루고, 어떤 연산이 필요한지 정의해보는 것
{: .prompt-info }

### 스택의 연산

핵심 연산: 삽입(push), 삭제(pop) → 스택의 상태 변화시킨다.

- push(e) : 새로운 요소 e를 스택의 맨 위에 추가
- pop() : 스택의 맨 위에 있는 요소를 꺼내서 반환

추가적인 연산: 스택의 상태(비어있는지 or 꽉 차있는지)를 검사, 스택 상단 요소 확인 → 스택의 상태 변화시키지 않는다.

- isEmpty() : 스택이 비어있으면 True, 아니면 False를 반환
- isFull() : 스택이 가득 차 있으면 True, 아니면 False를 반환
- peek() : 스택의 맨 위에 있는 항목을 삭제하지 않고 반환
- size() : 스택에 들어있는 전체 요소의 수를 반환

![스택의 일련의 연산](/assets/img/posts/stack-chapter1/stack-operations.png)

**오버플로(overflow)**는 포화 상태인 스택에 새로운 요소를 삽입하는 경우 발생하는 오류이다.

반대로 **언더플로(underflow)**는 공백 상태의 스택에서 pop()이나 peek()을 호출하면 삭제나 참조가 불가능하므로 발생하는 오류이다.

## 2. 배열 구조로 스택 구현하기

배열 구조 : 자료들을 배열에 모아 저장하는 방법, 모든 요소는 인접한 메모리 공간에 저장, 크기 고정

연결된 구조 : 흩어져있는 요소들을 연결, 추가 및 삭제가 쉽지만 관리 복잡

### 배열 구조의 스택을 위한 데이터

배열(array) : 요소들을 저장할 공간 → array[]

용량(capacity) : 이 공간의 크기(상수) 

top : 상단 요소의 위치(변수, 인덱스 저장)

![배열을 이용한 스택의 구조](/assets/img/posts/stack-chapter1/array-stack-structure.png)

```python
capacity = 10
array = [None] * capacity
top = -1
```

### 스택의 연산

- isEmpty()
    
    top이 -1이면 스택이 공백임을 이용해 스택이 비었는지를 확인한다.
    
    ```python
    def isEmpty():
    	if top == -1: return True  # 공백이면 True
    	else: return False         # 아니면 False
    ```
    
    간결한 방법
    
    ```python
    def isEmpty(): return top == -1  # 비교연산(==) 결과를 바로 반환
    ```
    
- isFull()
    
    top이 capacity - 1이면 포화 상태임을 이용해 스택이 가득 차 있는지를 확인한다.
    
    ```python
    def isFull(): return top == capacity - 1
    ```
    
- push(e)
    
    조건: 스택이 포화 상태가 아닐 때
    
    top을 하나 증가시키고 그 위치에 삽입
    
    ```python
    def push(e):
    	# global top
    	if not isFull():
    		top += 1
    		array[top] = e
    	else:
    		print("stack overflow")
    		exit()
    ```
    
- pop()
    
    조건: 스택이 공백 상태가 아닐 때
    
    top을 하나 감소시키고 이전 위치(top+1)의 요소 반환한다.
    
    ```python
    def pop(e):
    	# global top
    	if not isEmpty():
    		top -= 1
    		return array[top + 1]
    	else:
    		print("stack underflow")
    		exit()
    ```
    
- peek()
    
    조건: 스택이 공백 상태가 아닐 때
    
    top 위치의 요소 반환한다. (상단 요소 확인)
    
    ```python
    def peek():
    	if not isEmpty():
    		return array[top]
    	else: pass           # underflow 예외 처리 생략
    ```
    
- size()
    
    현재 스택의 요소 수 = top + 1
    
    ```python
    def size(): return top + 1
    ```
    

### 문제점

- top을 전역 변수로 인식하지 못해 생기는 문제
    
    → `global top` 문장을 각 함수에 추가하기
    
- ⚠️ 여러 개의 스택이 동시에 필요한 문제에 사용할 수 없는 문제
    
    → 스택을 클래스로 구현하기
    

### 스택을 클래스로 구현

```python
class ArrayStack:
	def __init__(self, capacity):
		self.capacity = capacity
		self.array = [None] * self.capacity
		self.top = -1
```

멤버변수들은 생성자 함수(`__init__()`)에서 선언하고 초기화한다.

생성자는 클래스의 객체가 만들어질 때마다 자동호출된다.

파이썬에서는 모든 멤버함수의 첫 번째 매개변수로 스택 객체 자신을 나타내는 `self` 를 넣기로 약속

- 스택 클래스의 연산
    
    ```python
    def isEmpty(self): return self.top == -1
    def isFull(self): return self.top == self.capacity-1
    
    def push(self, item):
    	if not self.isFull():
    		self.top += q
    		self.array[self.top] = item
    	else: pass # overflow 예외 생략
    	
    def pop(self):
    	if not self.isEmpty():
    		self.top -= 1
    		return self.array[self.top+1]
    		
    def peek(self):
    	if not self.isEmpty():
    		return self.array[self.top]
    	else: pass # underflow 예외 생략
    	
    def size(): return top+1
    ```
    

## 3. 스택의 응용: 괄호 검사

스택 문제 중 매우 중요한 유형

`{ A[(i+1)] = 0; }` 처럼 괄호가 정상적으로 짝지어졌는지 검사한다.

파일에서는 올바른 괄호 조건을 세 가지로 설명한다.

1. 왼쪽 괄호와 오른쪽 괄호
2. 같은 종류에서 왼쪽 괄호가 먼저 나와야 함
3. 다른 종류의 괄호 쌍이 교차하면 안 됨 

알고리즘

```
문자를 하나씩 검사한다.

열린 괄호 ( [ {
→ push

닫힌 괄호 ) ] }
→ pop
→ 짝이 맞는지 검사
```

예를 들어 `{ [ ( ) ] }` 일 때,

```
{ push
[ push
( push

) → ( pop
] → [ pop
} → { pop

stack empty
→ 정상
```

### 의사코드

```python
for ch in statement:
	if 여는 괄호:
		push(ch)
	
	elif 닫는 괄호:
		if stack이 비어있음:
			return False
		
		left = pop()
		
		if left와 ch의 종류가 다름:
			return False
		
return stack이 비어있는가
```

### 괄호 검사의 핵심

가장 최근에 열린 괄호가 가장 먼저 닫혀야 한다. → Last Open First Close 이므로 LIFO인 스택을 사용한다.

## 4. 파이썬에서 스택 사용하기

가장 간단한 방법은 list를 stack처럼 쓰는 것이다.

```python
stack = []

stack.append(10) # push
stack.append(20)

stack.pop() # pop
stack[-1]   # peek 
len(stack)  # size
len(stack) == 0  # isEmpty
```

리스트의 뒤쪽을 top으로 사용하는 것이 효율적이다.

| Stack | Python list |
| --- | --- |
| push | append() |
| pop | pop() |
| peek | s[-1] |
| size | len(s) |
| isEmpty | len(s) == 0 |

### queue 모듈의 LifoQueue 사용하기

파이썬 queue 모듈에서도 스택을 제공한다.

```python
import queue
s = queue.LifoQueue(maxsize=20)  # maxsize=0 이면 용량제한 없음
```

| ArrayStack | LifoQueue |
| --- | --- |
| push() | put() |
| pop() | get() |
| isEmpty() | empty() |
| isFull() | full() |
| peek() | 제공하지 않음 |

## 5. 시스템 스택과 순환호출

시스템 스택은 함수를 호출할 때 컴퓨터는 함수 호출에 필요한 정보를 시스템 스택에 저장한다. 
대표적으로 복귀 주소, 매개변수, 지역변수 등을 관리한다.

시스템 스택도 LIFO로 동작한다.

![함수 호출과 반환 과정의 시스템 스택 변화](/assets/img/posts/stack-chapter1/system-stack.png)

### 순환이란?

순환 또는 재귀(recursion)는 함수가 자기자신을 다시 호출하는 방법이다.

ex) 팩토리얼 : n! = n $\times$ (n-1)!

```python
def factiorial(n):
	if n == 1:
		return 1
	
	else:
		return n * factorial(n-1)
```

### 재귀에서 반드시 기억할 두 가지

1. 문제 크기가 줄어들어야 한다.
    
    `factorial(n-1)` 처럼  호출할수록 종료지점에 가까워져야 한다.
    
2. 종료조건이 있어야 한다.
    
    ```python
    if n == 1:
    	return 1
    ```
    
    이 조건이 없으면 계속 호출되고 결국 시스템 스택을 소진한다.
    

### 반복문 vs 재귀

팩토리얼에서는 반복 구조와 재귀 구조 모두 곱셈 횟수가 n-1번이다.

하지만 재귀는 “함수 호출 + 시스템 스택 사용”이라는 추가 비용이 있기 때문에 일반적으로 반복 구조보다 느릴 수 있다.

그럼에도 트리, 이진 탐색, 정렬처럼 재귀로 표현했을 때 훨씬 명확해지는 문제가 존재한다.

### 하노이의 탑

하노이의 탑은 재귀의 대표적인 예다.

![하노이의 탑 문제](/assets/img/posts/stack-chapter1/hanoi-tower.png)

n개의 원판을 A에서 C로 옮기려면 다음과 같이 나눈다.

1. n-1개를 A → B (위의 원판 n-1개를 A → B로 옮긴다)
2. 가장 큰 원판을 A → C (가장 큰 원판 1개를 A → C로 옮긴다)
3. n-1개를 B → C (B에 있는 원판 n-1개를 B → C로 옮긴다)

### 코드 핵심 구조

n : 원판 수

fr : 시작 막대

tmp : 임시 막대

to : 목표 막대

```python
def hanoi_tower(n, fr, tmp, to):
	if n == 1:       # 순환 호출을 멈추는 부분, 원판이 하나라면 바로 이동
		move fr → to
	
	else:
		hanoi_tower(n-1, fr, to, tmp)  # 단계 1
		move n fr → to                 # 단계 2
		hanoi_tower(n-1, tmp, fr, to)  # 단계 3
```

완성 코드

```python
# 원판 n개를 fr에서 tmp를 임시로 사용해서 to로 옮겨라
def hanoi_tower(n, fr, tmp, to):
	if (n == 1):
		print("원판 1: %s --> %s" % (fr, to))
		
	else:
		hanoi_tower(n-1, fr, to, tmp)
		print("원판 %d: %s --> %s" % (n, fr, to))
		hanoi_tower(n-1, tmp, fr, to)
```
