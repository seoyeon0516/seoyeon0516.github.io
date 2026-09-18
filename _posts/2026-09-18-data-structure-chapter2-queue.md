---
title: "[자료구조와 알고리즘 with 파이썬] Chapter 02 큐"
date: 2026-09-18 00:00:00 +0900
categories: [Study, Data Structure]
tags: [data-structure, queue, deque, python]
description: 큐의 개념과 배열 구현, 원형 큐와 링 버퍼, 덱과 상속, 파이썬 표준 모듈 활용 정리
---

## 1. 큐란?

큐(queue)는 가장 먼저 들어온 데이터가 가장 먼저 나오는 자료구조다.

FIFO(First-In First-Out, 선입선출) 

front : 전단, 삭제 쪽

rear : 후단, 삽입 쪽

![큐의 구조](/assets/img/posts/queue-chapter2/queue-structure.png)

**큐가 필요한 이유**

데이터가 처리 속도보다 빠르게 들어오는 경우 일시적으로 데이터를 보관하는 버퍼(buffer)로 사용할 수 있다.

### 큐의 연산

| 연산 | 의미 |
| --- | --- |
| enqueue(e) | 후단에 삽입 |
| dequeue() | 전단 요소 제거 후 반환 |
| peek() | 전단 요소 확인 |
| isEmpty() | 공백 검사 |
| isFull() | 포화 검사 |
| size() | 요소 개수 |

큐에서도 포화 상태에서 `enqueue()` 하면 overflow, 공백 상태에서 `dequeue()` 나 `peek()` 하면 underflow가 발생한다.

## 2. 배열로 구현하는 큐

배열로 큐를 구현하려면 array, capacity, front(첫 번째 요소 바로 이전 위치), rear(마지막 요소의 위치)가 필요하다.

여기서 front가 **첫 요소 자체가 아니라 첫 요소의 바로 앞이라는 점**이 중요하다.

### 선형 큐의 문제점

단순 배열 큐에서는 rear가 계속 오른쪽으로 이동한다.

앞쪽에는 공간이 있지만 rear는 배열 끝에 있으므로 새로운 요소를 바로 삽입할 수 없다.

결국 모든 데이터를 앞으로 이동시켜야 한다.

이 방식이 **선형 큐(linear queue)**이며 요소 이동이 필요해서 비효율적이다.

![선형 큐의 문제점: 요소들을 앞으로 이동해야 새 요소를 삽입할 수 있음](/assets/img/posts/queue-chapter2/linear-queue-problem.png)

### 원형 큐

해결 방법은 배열을 원처럼 연결되어 있다고 생각하는 것이다.

실제로 배열이 원형인 것은 아니고 front, rear 인덱스를 회전시킨다.

핵심 공식 : `(index + 1) % capacity`

```python
front = (front + 1) % capacity
rear = (rear + 1) % capacity
```

- 초기 상태 : `front = 0` , `rear = 0`
- 공백 상태 : `front == rear`
- 포화 상태
    
    문제 : 모든 공간을 사용해 rear가 한바퀴 돌아 front와 같아지면 `front == rear` 이므로 공백 상태와 구분할 수 없다. 
    
    그래서 한 칸을 항상 비워둔다. 따라서 포화 상태는 `front == (rear + 1) % capacity` 이다.
    
    → capacity = 8 이면 실제 저장 가능한 데이터는 7개이다.
    
- enqueue()
    
    삽입 과정
    
    1. 포화 상태 확인
    2. rear 한 칸 회전
    3. 해당 위치에 저장
    
    ```python
    self.rear = (self.rear + 1) % self.capacity
    self.array[self.rear] = item
    ```
    
- dequeue()
    
    삭제 과정
    
    1. 공백 상태 확인
    2. front 한 칸 회전
    3. 해당 위치 값 반환
    
    ```python
    self.front = (self.front + 1) % capacity
    return self.array[self.front]
    ```
    
- peek()
    
    front는 첫 요소의 바로 이전 위치를 가리키는 것을 유의한다.
    
    단, front 자체를 변경하면 안된다.
    
    ```python
    self.array[(self.front + 1) % self.capacity]
    ```
    
- size()
    
    선형 큐라면 `rear - front` 로 계산하면 되지만 원형 큐에서는 `rear < front` 인 상황이 존재한다.
    
    ⭐️ 그래서 `rear - front + capacity) % capacity` 을 사용한다. ⭐️
    

### 원형 큐 핵심 공식 모음

```python
# 공백
front == rear

# 포화
front == (rear + 1) % capacity

# rear 이동
rear = (rear + 1) % capacity

# front 이동
front = (front + 1) % capacity

# 요소 수
(rear - front + capacity) % capacity
```

### 원형 큐를 링 버퍼로 사용하기

원형 큐는 최근 데이터만 유지하는 버퍼(ring buffer)로도 활용할 수 있다.

예를 들어 최대 7개를 보관한다고 할 때, 

```python
[0 1 2 3 4 5 6]
```

가득 찬 상태에서 7이 들어오면 가장 오래된 0을 버린다.

```python
[1 2 3 4 5 6 7]
```

**핵심**

일반 큐는 포화 상태에서 enqueue()를 하면 overflow가 되지만, 링 버퍼는 가장 오래된 데이터를 삭제하고 새 데이터를 저장한다.

→ `front == rear` 가 되면 front를 한 칸 이동시켜 가장 오래된 데이터를 제거한다.

## 3. 덱이란?

덱(deque)은 double-ended queue의 줄임말이다.

큐와 달리 전단(front), 후단(rear) 양쪽에서 모두 삽입과 삭제가 가능하다.

단, 중간 삽입·삭제는 불가능하다.

### 덱의 연산

| 연산 | 의미 | 동일한 큐 연산 | 동일한 스택 연산 |
| --- | --- | --- | --- |
| addFront(e) | 전단에 삽입 |  |  |
| addRear(e)  | 후단에 삽입 | enqueue() | push() |
| deleteFront() | 전단 요소를 꺼내서 반환 | dequeue() |  |
| deleteRear() | 후단 요소를 꺼내서 반환 |  | pop() |
| getFront() | 전단 요소를 삭제하지 않고 반환 | peek() |  |
| getRear() | 후단 요소를 삭제하지 않고 반환 |  | peek() |
| isEmpty() | 공백 검사 |  |  |
| isFull() | 포화 검사 |  |  |
| size() | 요소의 개수 |  |  |

덱 하나로 스택과 큐를 모두 표현할 수 있다.

### 반시계 방향 이동

원형 덱에서는 양쪽을 모두 사용하므로 인덱스를 감소시키는 경우도 있다.

전단 회전(반시계 방향) : `front = (front - 1 + capacity) % capacity`

후단 회전(시계 방향) : `rear = (rear - 1 + capacity) % capacity` 

> 💡 (+ capacity)를 먼저 하는 이유는 음수 인덱스를 피하면서 원형으로 회전시키기 위함
{: .prompt-info }

## 4. 상속을 이용한 덱의 구현

원형 덱과 원형 큐는 구조가 매우 비슷하다.

따라서 원형 큐 ArrayQueue를 다시 처음부터 복사해 구현하는 대신 상속(inheritance)을 이용한다.

### 상속 구조

부모 클래스 : ArrayQueue

자식 클래스 : CircularDeque

```python
class CircularDeque(ArrayQueue):
	def __init__(self, capacity = 10):
		super().__init__(capacity)
```

### 부모 클래스의 기능 재사용

isEmpty(), isFull(), size(), display() 연산 그대로 사용 가능하다.

또 다음처럼 이름만 다른 기능도 재사용한다.

```python
def addRear(self, item):
	self.enqueue(item)

def deleteFront(self):
	return self.dequeue()
	
def getFront(self):
	return self.peek()
```

### 덱에서 새로 구현해야 하는 것

큐에 없었던 addFront(), deleteRear(), getRear()만 새로 작성하면 된다.

### addFront()

전단에 데이터를 저장하고 front를 반시계 방향으로 이동한다.

```python
self.array[self.front] = item
self.front = (self.front - 1 + self.capacity) % self.capacity
```

### deleteRear()

현재 rear 요소를 저장해두고 rear를 반시계 방향으로 이동시킨 뒤 저장했던 값을 반환한다.

```python
item = self.array[self.rear]
self.rear = (self.rear - 1 + self.capacity) % self.capacity
```

## 5. 파이썬에서 큐와 사용하기

실제 파이썬에서는 직접 만든 ArrayQueue, CircularDeque 대신 표준 모듈을 사용할 수 있다.

### queue 모듈의 Queue 사용하기

```python
import queue

q = queue.Queue(maxsize=20) # maxsize=0 이면 용량 제한이 없는 큐
```

대응관계

| ArrayQueue | queue.Queue |
| --- | --- |
| enqueue() | put() |
| dequeue() | get() |
| isEmpty() | empty() |
| isFull() | full() |
| peek() | 제공하지 않음 |

### collections 모듈의 deque 클래스 사용하기

```python
import collections

dq = collections.deque()
```

주요 연산

| 의미 | collections.deque |
| --- | --- |
| 전단 삽입 | appendleft() |
| 전단 삭제 | popleft() |
| 후단 삽입 | append() |
| 후단 삭제 | pop() |

## Chapter 01~02 복습 메모

- Stack
    - 마지막에 들어간 것이 먼저 나온다.
    - LIFO, push / pop / peek
    - Undo, 괄호 검사, 함수 호출, 재귀에서 사용
- Queue
    - 먼저 들어간 것이 먼저 나온다.
    - FIFO, enqueue / dequeue
    - front에서 삭제하고 rear에서 삽입
- Circular Queue
    - 배열의 빈 공간을 다시 활용하기 위해 인덱스를 원형으로 회전시킨다.
    - 핵심은 `% capacity`
- Deque
    - 큐의 양쪽 끝에서 삽입·삭제할 수 있다.
    - Stack과 Queue의 기능을 모두 표현할 수 있다.
- Recursion
    - 함수가 자기 자신을 호출한다.
    - 호출할수록 문제의 크기가 작아져야 하며 종료 조건이 반드시 필요하다.
    - 내부적으로 시스템 스택이 사용된다.
