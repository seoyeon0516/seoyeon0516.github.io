---
title: "프로그래머스 고득점 Kit 문제 풀이 01"
date: 2026-10-05 00:00:00 +0900
categories: [Study, Algorithm]
tags: [programmers, python, algorithm, hash, queue, heap]
description: 프로그래머스 알고리즘 고득점 Kit의 전화번호 목록, 프로세스, 더 맵게 문제 풀이
math: true
---

[프로그래머스 알고리즘 고득점 Kit](https://school.programmers.co.kr/learn/challenges?tab=algorithm_practice_kit)

## 해시 - 전화번호 목록

> **문제 요약**
> 전화번호부에 저장된 번호 중에서 어떤 번호가 다른 번호의 접두어가 되는지 확인하는 문제이다. 예를 들어 `119`와 `1195524421`이 함께 있다면 `119`가 다른 번호의 시작 부분과 일치하므로 `False`를 반환한다. 접두어 관계가 하나도 없다면 `True`를 반환한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42577)

1. 입력: 전화번호들이 들어 있는 문자열 배열 `phone_book`
2. 출력: 어떤 전화번호가 다른 전화번호의 접두어이면 False, 그렇지 않으면 True
3. 내가 손으로 풀면:
   - 각 전화번호의 앞부분을 한 글자씩 늘려가며 자른다.
   - 자른 접두어가 실제 전화번호 목록에 존재하는지 확인한다.
4. 반복해야 하는 것:
   - 전화번호를 하나씩 검사
   - 각 번호의 접두어를 하나씩 검사
5. 조건: 자기 자신 전체는 검사하지 않는다.
6. 기억해야 하는 값: 전화번호가 저장된 집합
7. 예상 시간복잡도: 전화번호 개수를 N, 최대 길이를 L이라고 할 때 $O(N \times L^2)$

set에서 `in` 검색은 평균적으로 $O(1)$이다.

> 해시 사고방식: `set()` 문법 자체보다 **어떤 값이 존재하는지를 계속 찾아야 한다 → 미리 해시에 넣어두고 빠르게 검색하자**라고 생각한다.

```python
def solution(phone_book):
    answer = True

    phone_set = set(phone_book)
    for number in phone_book:
        for i in range(1, len(number)):
            prefix = number[:i]

            if prefix in phone_set:
                return False

    return answer
```

## 스택/큐 - 프로세스

> **문제 요약**
> 실행 대기 큐의 맨 앞 프로세스를 하나씩 꺼내되, 대기 중인 프로세스 가운데 더 높은 우선순위가 있다면 현재 프로세스를 다시 큐의 뒤로 보낸다. 현재 프로세스보다 우선순위가 높은 프로세스가 없을 때만 실행하며, 처음 `location` 위치에 있던 프로세스가 몇 번째로 실행되는지 구하는 문제이다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42587)

1. 입력:
   - `priorities`: 각 프로세스의 우선순위
   - `location`: 내가 찾는 프로세스의 위치
2. 출력: 내가 찾는 프로세스가 몇 번째로 실행되는지
3. 내가 손으로 풀면:
   - 맨 앞 프로세스를 꺼낸다.
   - 남아 있는 프로세스들의 우선순위를 확인한다.
   - 나보다 우선순위가 높은 프로세스가 하나라도 있으면 꺼낸 프로세스를 맨 뒤에 넣는다.
   - 더 높은 프로세스가 없으면 실행한다.
4. 반복해야 하는 것:
   - 맨 앞 프로세스를 꺼내고 남은 프로세스 중 더 높은 우선순위가 있는지 확인하는 과정을 반복한다.
5. 조건:
   - 더 높은 우선순위 프로세스가 있으면 → 현재 프로세스를 큐의 뒤로 보냄
   - 더 높은 우선순위가 없으면 → 현재 프로세스를 실행하고 실행 횟수 1 증가
   - 실행한 프로세스의 원래 위치가 `location`이면 → 실행 횟수 반환
6. 기억해야 하는 값:
   - 각 프로세스의 원래 위치(index)
   - 각 프로세스의 우선순위(priority)
   - 지금까지 실행된 프로세스의 개수(answer)
7. 예상 시간복잡도: $O(n^2)$

`enumerate(리스트)`는 각 요소를 `(위치, 값)` 형태로 보여준다.

```python
def solution(priorities, location):
    answer = 0
    queue = list(enumerate(priorities))

    while queue:
        current = queue.pop(0)
        index, priority = current

        higher = False

        for idx, p in queue:
            if p > priority:
                higher = True
                break

        if higher:
            queue.append(current)
        else:
            answer += 1

            if index == location:
                return answer
```

### 큐 문법 사용 버전

효율성 차이가 있다.

`list.pop(0)`은 맨 앞을 없앤 뒤 나머지 원소들을 앞으로 당겨야 해서 $O(N)$이고, `deque.popleft()`는 바로 맨 앞을 제거해서 $O(1)$이다.

```python
from collections import deque


def solution(priorities, location):
    answer = 0
    queue = deque(enumerate(priorities))  # list() 대신

    while queue:
        current = queue.popleft()  # pop(0) 대신
        index, priority = current

        higher = False

        for idx, p in queue:
            if p > priority:
                higher = True
                break

        if higher:
            queue.append(current)
        else:
            answer += 1

            if index == location:
                return answer
```

## 힙 - 더 맵게

> **문제 요약**
> 모든 음식의 스코빌 지수를 `K` 이상으로 만들기 위해 가장 맵지 않은 음식 두 개를 반복해서 섞는 문제이다. 새 음식의 스코빌 지수는 `가장 작은 값 + 두 번째로 작은 값 × 2`로 계산한다. 모든 음식이 기준을 만족할 때까지 섞은 최소 횟수를 반환하며, 기준을 만족시킬 수 없다면 `-1`을 반환한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42626)

힙을 쓰는 이유는 현재 음식 중 가장 작은 값 2개를 계속 꺼내야 하기 때문이다. 최솟값을 빠르게 꺼내는 자료구조인 **최소 힙(min heap)**을 사용한다.

1. 입력:
   - 각 음식의 스코빌 지수가 담긴 리스트 `scoville`
   - 목표로 하는 기준 스코빌 지수 `K`
2. 출력: 모든 음식의 스코빌 지수를 K 이상으로 만들기 위해 섞어야 하는 최소 횟수
3. 내가 손으로 풀면:
   - 가장 작은 두 값을 꺼낸다.
   - 두 값을 공식에 따라 섞는다.
   - 섞은 값을 다시 넣는다.
   - 최솟값이 K 이상일 때까지 반복한다.
4. 반복해야 하는 것: 최솟값이 K 이상이 될 때까지 가장 작은 두 값을 꺼내고 다시 섞는 과정
5. 조건:
   - 최솟값이 K 이상이면 → 성공
   - K보다 작은데 음식을 더 이상 2개 꺼낼 수 없으면 → 실패(-1)
6. 기억해야 하는 값: 섞은 횟수 `answer`
7. 예상 시간복잡도: $O(N \log N)$
   - `heapq.heappop(scoville)` / `heapq.heappush(scoville, x)` → $O(\log N)$

```python
import heapq


def solution(scoville, K):
    answer = 0
    heapq.heapify(scoville)

    while scoville[0] < K:
        if len(scoville) < 2:
            return -1

        first = heapq.heappop(scoville)
        second = heapq.heappop(scoville)

        new_scoville = first + (second * 2)

        heapq.heappush(scoville, new_scoville)
        answer += 1

    return answer
```

- `append` + `heapify` → 전체를 다시 힙으로 정리 → $O(N)$
- `heappush` → 새 값 하나만 적절한 위치로 정리 → $O(\log N)$

따라서 `heappush`가 훨씬 효율적이다.
