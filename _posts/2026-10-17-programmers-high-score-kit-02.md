---
title: "프로그래머스 고득점 Kit 문제 풀이 02"
date: 2026-10-17 00:00:00 +0900
categories: [Study, Algorithm]
tags: [programmers, python, algorithm, sorting]
description: 프로그래머스 알고리즘 고득점 Kit의 K번째수, 가장 큰 수, H-Index 문제 풀이
math: true
---

[프로그래머스 알고리즘 고득점 Kit](https://school.programmers.co.kr/learn/challenges?tab=algorithm_practice_kit)

## 정렬 - K번째수

> **문제 요약**
> 배열에서 `i`번째부터 `j`번째까지 자른 뒤 정렬하여 `k`번째 수를 구한다. 여러 명령에 대해 이 과정을 반복하고 결과를 배열로 반환한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42748)

1. 입력:
   - 원본 배열 `array`
   - `[i, j, k]`를 원소로 가진 2차원 배열 `commands`
2. 출력: 각 명령에서 찾은 `k`번째 수를 담은 배열
3. 내가 손으로 풀면:
   - 명령 하나를 꺼내 `i`번째부터 `j`번째까지 자른다.
   - 자른 배열을 오름차순으로 정렬한다.
   - 정렬한 배열의 `k`번째 수를 결과에 넣는다.
4. 반복해야 하는 것: `commands`의 각 명령을 순서대로 처리한다.
5. 조건: 문제의 `i`, `j`, `k`는 1부터 세지만, Python 배열 인덱스는 0부터 센다.
6. 기억해야 하는 값:
   - `start`, `end`, `k`: 현재 명령의 세 값
   - `temp`: 잘라서 정렬한 임시 배열
   - `answer`: 각 명령의 결과를 저장할 배열
7. 예상 시간복잡도: 명령 수를 $q$, 한 번에 자르는 구간의 최대 길이를 $m$이라고 하면 $O(qm\log m)$
   - 슬라이싱: 명령당 $O(m)$
   - 정렬: 명령당 $O(m\log m)$
   - 추가 공간복잡도: 임시 배열과 결과 배열을 합쳐 $O(m+q)$

예를 들어 `[1, 5, 2, 6, 3, 7, 4]`에 `[2, 5, 3]`을 적용하면 `[5, 2, 6, 3]`을 잘라 정렬한 뒤 `[2, 3, 5, 6]`의 세 번째 값인 `5`를 선택한다.

`array[start - 1:end]`에서 시작점만 1을 빼는 이유는 Python 슬라이스의 끝 인덱스가 결과에 포함되지 않기 때문이다.

```python
def solution(array, commands):
    answer = []

    for start, end, k in commands:
        temp = array[start - 1:end]
        temp.sort()
        answer.append(temp[k - 1])

    return answer
```

## 정렬 - 가장 큰 수

> **문제 요약**
> 0 이상의 정수들을 이어 붙여 만들 수 있는 가장 큰 수를 문자열로 반환한다. 숫자 자체의 크기가 아니라 두 문자열을 이어 붙인 결과를 비교해야 한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42746)

1. 입력: 0 이상의 정수가 담긴 배열 `numbers`
2. 출력: 주어진 숫자를 이어 붙여 만들 수 있는 가장 큰 수를 문자열로 반환
3. 내가 손으로 풀면:
   - 모든 숫자를 문자열로 바꾼다.
   - 두 문자열 `a`, `b`를 골라 `a + b`와 `b + a`를 비교한다.
   - 더 큰 연결 결과가 나오는 순서대로 정렬한다.
   - 정렬된 문자열을 모두 이어 붙인다.
4. 반복해야 하는 것: 정렬 과정에서 두 문자열의 연결 결과를 반복 비교한다.
5. 조건:
   - `a + b > b + a`이면 `a`를 앞에 놓는다.
   - `a + b < b + a`이면 `b`를 앞에 놓는다.
   - 모든 값이 0이면 결과는 `"000..."` 대신 `"0"`이다.
6. 기억해야 하는 값:
   - `numbers`: 문자열로 변환한 숫자 배열
   - `a`, `b`: 정렬 중 비교하는 두 문자열
   - `compare`: 두 문자열의 순서를 정하는 비교 함수
   - `answer`: 정렬된 문자열을 이어 붙인 결과
7. 예상 시간복잡도: 숫자 개수를 $n$, 최대 자릿수를 $d$라고 하면 $O(n\log n\cdot d)$
   - 문자열 변환과 연결: 각각 $O(nd)$
   - 정렬: 비교 한 번에 최대 $O(d)$가 들어 $O(n\log n\cdot d)$
   - 추가 공간복잡도: 문자열 배열과 정렬 보조 공간을 포함해 $O(nd)$

예를 들어 `[3, 30, 34, 5, 9]`에서 `"3" + "30" = "330"`은 `"30" + "3" = "303"`보다 크므로 `3`을 `30`보다 앞에 둔다. `"34" + "3" = "343"`도 `"3" + "34" = "334"`보다 크므로 `34`를 `3`보다 앞에 둔다. 최종 순서는 `9, 5, 34, 3, 30`이고 결과는 `"9534330"`이다.

```python
from functools import cmp_to_key


def compare(a, b):
    if a + b > b + a:
        return -1
    elif a + b < b + a:
        return 1
    return 0


def solution(numbers):
    numbers = list(map(str, numbers))
    numbers.sort(key=cmp_to_key(compare))

    answer = ''.join(numbers)
    return '0' if answer[0] == '0' else answer
```

**핵심:** 일반 내림차순 정렬이 아니라 문자열을 이어 붙인 결과를 기준으로 정렬한다. `cmp_to_key(compare)`는 두 값을 비교하는 함수를 `sort()`의 정렬 기준으로 바꾼다. 비교 함수가 `-1`을 반환하면 `a`가 앞에, `1`을 반환하면 `b`가 앞에 온다.

## 정렬 - H-Index

> **문제 요약**
> 논문별 인용 횟수에서 `h`번 이상 인용된 논문이 `h`편 이상이 되는 가장 큰 `h`를 구한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/42747)

1. 입력: 논문별 인용 횟수가 담긴 배열 `citations`
2. 출력: 연구자의 H-Index (정수)
3. 내가 손으로 풀면:
   - 인용 횟수를 내림차순으로 정렬한다.
   - 인용 횟수가 많은 논문부터 순위를 1부터 매겨 확인한다.
   - 현재 논문의 인용 횟수가 순위 이상이면 그 순위를 후보값으로 갱신한다.
   - 조건을 만족하지 않으면 반복을 멈춘다.
4. 반복해야 하는 것: 각 논문의 순위와 인용 횟수를 차례대로 비교한다.
5. 조건: 현재 인용 횟수가 순위보다 작으면 그 뒤의 논문도 조건을 만족할 수 없다.
6. 기억해야 하는 값:
   - `citations`: 내림차순 정렬한 인용 횟수 배열
   - `i`: 현재 논문의 순위 (1부터 시작)
   - `count`: 현재 논문의 인용 횟수
   - `answer`: 지금까지 조건을 만족한 최대 순위
7. 예상 시간복잡도: 정렬 $O(n\log n)$과 순회 $O(n)$을 합쳐 $O(n\log n)$
   - 추가 공간복잡도: 입력 배열 자체를 제외하고 Python 정렬의 보조 메모리까지 고려하면 최악의 경우 $O(n)$

예를 들어 `[3, 0, 6, 1, 5]`를 내림차순 정렬하면 `[6, 5, 3, 1, 0]`이 된다.

| 순위 `i` | 인용 횟수 | `count >= i` | 판정 |
| --- | --- | --- | --- |
| 1 | 6 | `6 >= 1` | 만족 |
| 2 | 5 | `5 >= 2` | 만족 |
| 3 | 3 | `3 >= 3` | 만족 |
| 4 | 1 | `1 < 4` | 불만족 |

따라서 H-Index는 `3`이다.

```python
def solution(citations):
    citations.sort(reverse=True)
    answer = 0

    for i, count in enumerate(citations, start=1):
        if count >= i:
            answer = i
        else:
            break

    return answer
```

**핵심:** 인용 횟수 자체가 H-Index가 아니라, 인용 횟수가 논문 순위 이상인 최대 순위가 H-Index다. `enumerate(citations, start=1)`로 논문 순위를 1부터 매긴다.

## 코딩테스트에서 기억해야 할 접근법

- K번째수: 배열의 특정 구간을 정렬하고 순위를 찾는다 → 슬라이싱 + 정렬
- 가장 큰 수: 단순 오름차순·내림차순으로 풀 수 없다 → 문자열 연결 결과로 비교 기준 정의
- H-Index: 특정 조건을 만족하는 최댓값을 찾는다 → 정렬 후 순위와 값 비교
