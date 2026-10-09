---
title: "재귀 문제 풀이 모음 01"
date: 2026-09-29 00:10:00 +0900
categories: [Study, Algorithm]
tags: [programmers, python, algorithm, recursion]
description: 프로그래머스 하노이의 탑, 괄호 변환, 유사 칸토어 비트열 문제 풀이
math: true
---

## 하노이의 탑

> **문제 요약**
> 1번 기둥에 쌓인 n개의 원판을 한 번에 하나씩 3번 기둥으로 옮기는 최소 이동 순서를 구하는 문제이다. 큰 원판을 작은 원판 위에 놓을 수 없으며, 각 이동은 `[출발 기둥, 도착 기둥]` 형태로 기록한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/12946)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/12946)

1. 입력: 원판의 개수 n
2. 출력: 원판을 옮기는 순서를 담은 2차원 배열
3. 내가 손으로 풀면:
   - 출발지 A, 임시 막대 B, 도착지 C라고 할 때
   - ① 위의 n-1개를 A → B로 옮긴다.
   - ② 가장 큰 원판을 A → C로 옮긴다.
   - ③ n-1개를 B → C로 옮긴다.
4. 반복해야 하는 것:
   - n개의 원판을 옮기는 과정에서 n-1개의 원판을 옮기는 동일한 문제를 반복해서 해결한다.
5. 조건:
   - 한 번에 원판 하나만 옮길 수 있다.
   - 큰 원판을 작은 원판 위에 놓을 수 없다.
   - 원판이 1개라면 바로 출발지 → 도착지로 옮길 수 있다.
6. 기억해야 하는 값:
   - 옮길 원판의 개수 n
   - 출발지
   - 임시 막대
   - 도착지
   - 실제 이동 순서
7. 예상 시간복잡도:
   - 이동 횟수 = 이전 이동 횟수 × 2 + 1 → $2^n - 1$

```python
def solution(n):
    answer = []

    def hanoi(n, fr, tmp, to):
        if n == 1:
            answer.append([fr, to])
            return

        hanoi(n-1, fr, to, tmp)
        answer.append([fr, to])
        hanoi(n-1, tmp, fr, to)

    hanoi(n, 1, 2, 3)

    return answer
```

## 괄호 변환

> **문제 요약**
> `(`와 `)`의 개수는 같지만 짝이 올바르지 않을 수 있는 균형잡힌 괄호 문자열을 주어진 재귀 절차에 따라 올바른 괄호 문자열로 변환하는 문제이다. 문자열을 더 이상 나눌 수 없는 균형잡힌 `u`와 나머지 `v`로 분리하는 과정이 핵심이다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/60058)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/60058)

1. 입력: 균형잡힌 괄호 문자열 p
2. 출력: 올바른 괄호 문자열
3. 내가 손으로 풀면:
   - 가장 작은 균형잡힌 u와 나머지 v로 분리
   - u가 올바르면 → u + 변환(v)
   - u가 올바르지 않으면
     - `(` + 변환(v) + `)`
     - u 양끝 제거
     - 나머지 괄호 방향을 뒤집어서 추가
4. 반복해야 하는 것:
   - v에 대해 동일한 괄호 변환을 반복
5. 조건:
   - 빈 문자열이면 그대로 반환
   - 균형잡힘: `(`와 `)` 개수가 같음
   - 올바름: 왼쪽부터 확인할 때 `)`가 `(`보다 많아지면 안 됨
6. 기억해야 하는 값:
   - 괄호 개수를 확인하는 count
   - 분리한 u, v
   - 변환 결과 result

```python
def solution(p):

    def correct(s):
        count = 0

        for ch in s:
            if ch == '(':
                count += 1
            else:
                count -= 1

            if count < 0:
                return False

        return True

    def convert(p):
        # 1. 빈 문자열이면 반환
        if p == '':
            return ''

        # 2. u, v 분리
        count = 0

        for i in range(len(p)):
            if p[i] == '(':
                count += 1
            else:
                count -= 1

            if count == 0:
                u = p[:i+1]
                v = p[i+1:]
                break

        # 3. u가 올바른 괄호 문자열
        if correct(u):
            return u + convert(v)

        # 4. u가 올바르지 않은 괄호 문자열
        else:
            result = '('
            result += convert(v)
            result += ')'

            for ch in u[1:-1]:
                if ch == '(':
                    result += ')'
                else:
                    result += '('

            return result

    return convert(p)
```

## 유사 칸토어 비트열

> **문제 요약**
> 0단계의 `1`에서 시작해 매 단계마다 1은 `11011`, 0은 `00000`으로 치환하여 만든 비트열에서 구간 `[l, r]`에 포함된 1의 개수를 구하는 문제이다. 전체 비트열을 직접 만들지 않고 각 위치가 0이 되는 구조를 추적해야 한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/148652)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/148652)

- n = 0 → `1`
- n = 1 → `11011`
- n = 2 → `11011 11011 00000 11011 11011`

큰 구조가 `1 1 0 1 1`이며, 데이터 있음은 1, 데이터 없음은 0이다.

따라서 n이 몇이든 전체를 크게 5등분하면 `있음 | 있음 | 전부 0 | 있음 | 있음`이 된다.

1. 입력: n, l, r
2. 출력: l번째부터 r번째까지 1의 개수
3. 내가 손으로 풀면:
   - 각 위치가 0인지 1인지 확인하고 1의 개수를 센다.
4. 반복해야 하는 것:
   - 현재 위치가 11011의 어느 위치인지 확인한다.
   - 이전 단계의 위치로 계속 올라간다.
5. 조건:
   - 11011에서 가운데(index 2)는 0이다.
   - 따라서 `i % 5 == 2`이면 해당 위치는 0이다.
6. 기억해야 하는 값:
   - 현재 확인하고 있는 위치 i
7. 시간복잡도:
   - $O((r-l+1) \times n)$

```python
def solution(n, l, r):
    answer = 0

    for i in range(l - 1, r):
        if is_one(i):
            answer += 1

    return answer

def is_one(i):
    while i > 0:
        if i % 5 == 2:
            return False

        i //= 5

    return True
```
