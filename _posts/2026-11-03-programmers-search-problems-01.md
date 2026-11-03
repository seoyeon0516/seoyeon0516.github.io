---
title: "프로그래머스 탐색 문제 풀이 01 — 모의고사·카펫·순위 검색·입국심사"
date: 2026-11-03 00:00:00 +0900
categories: [Study, Algorithm]
tags: [programmers, python, algorithm, brute-force, binary-search, problem-solving]
description: 모의고사·카펫·순위 검색·입국심사를 입력과 출력, 손풀이, 단순한 방법, 코드 동작 순서로 정리
math: true
---

문제를 읽고 바로 알고리즘 이름을 떠올리기보다, **입출력을 적고 → 예시를 손으로 풀고 → 가장 단순한 방법을 코드로 옮기는 순서**로 연습한다. 아래는 쉬운 문제부터 어려운 문제까지 네 문제를 같은 흐름으로 정리한 기록이다.

> 모의고사와 카펫은 프로그래머스에서 **완전탐색** 문제로 분류된다. 순위 검색과 입국심사에서는 **이진 탐색**을 활용한다. 여기서 '탐색'은 가능한 답을 확인하는 넓은 의미도 함께 사용했다.

## 1. 모의고사

[프로그래머스 문제 보기](https://school.programmers.co.kr/learn/courses/30/lessons/42840?language=python3){:target="_blank" rel="noopener noreferrer"}

### 입력과 출력을 한 문장으로 적기

정답 배열 `answers`를 입력받아 **가장 많이 맞힌 수포자의 번호**를 배열로 반환한다. 여러 명이면 번호를 오름차순으로 반환한다.

### 예시를 손으로 풀기

`answers = [1, 2, 3, 4, 5]`라면 1번 수포자는 자신의 패턴대로 다섯 문제를 모두 맞힌다. 2번과 3번도 각각 자신의 패턴을 정답과 비교하고, 세 사람의 점수 중 가장 높은 점수를 받은 번호를 고른다. 이 예시의 결과는 `[1]`이다.

### 가장 단순한 방법부터 생각하기

1. 수포자 세 명의 찍기 패턴과 점수 `[0, 0, 0]`을 준비한다.
2. 정답 배열을 처음부터 끝까지 순회한다.
3. 각 문제에서 세 수포자가 찍은 답을 정답과 비교해, 맞혔다면 해당 점수를 1 올린다.
4. 최고 점수를 구하고 그 점수와 같은 수포자의 번호를 결과에 추가한다.

패턴은 정답 배열보다 짧을 수 있다. 따라서 `i % len(pattern)`으로 패턴의 끝에 도달하면 다시 처음부터 사용한다.

### Python 코드

```python
def solution(answers):
    patterns = [
        [1, 2, 3, 4, 5],
        [2, 1, 2, 3, 2, 4, 2, 5],
        [3, 3, 1, 1, 2, 2, 4, 4, 5, 5],
    ]
    scores = [0, 0, 0]

    for i, correct_answer in enumerate(answers):
        for student in range(3):
            pattern = patterns[student]
            picked_answer = pattern[i % len(pattern)]

            if picked_answer == correct_answer:
                scores[student] += 1

    highest = max(scores)
    result = []

    for student in range(3):
        if scores[student] == highest:
            result.append(student + 1)

    return result
```

### 코드가 작동하는 순서

- `patterns[0]`, `patterns[1]`, `patterns[2]`는 각각 1·2·3번 수포자의 찍기 방식이다. `scores`의 같은 인덱스에 각자의 점수를 저장한다.
- `enumerate(answers)`는 문제의 인덱스 `i`와 그 문제의 정답 `correct_answer`를 함께 꺼낸다. 예를 들어 첫 문제에서는 `i=0`이다.
- 안쪽 `for student in range(3)`은 같은 문제에 대해 수포자 세 명을 차례로 확인한다.
- `i % len(pattern)`은 반복 패턴의 현재 위치다. 1번 패턴의 길이는 5이므로 6번째 문제(`i=5`)에서는 `5 % 5 = 0`, 다시 첫 번째 찍기 답을 사용한다.
- 정답과 찍은 답이 같을 때만 `scores[student]`를 올린다.
- `max(scores)`로 최고 점수를 찾고, 동점인 모든 수포자를 결과에 넣는다. 인덱스는 0부터, 수포자 번호는 1부터 시작하므로 `student + 1`을 넣는다. 작은 번호부터 확인하므로 결과도 오름차순이다.

**시간 복잡도:** 정답 수를 $n$이라 하면 각 문제에서 세 명만 비교하므로 $O(n)$이다.

## 2. 카펫

[프로그래머스 문제 보기](https://school.programmers.co.kr/learn/courses/30/lessons/42842?language=python3){:target="_blank" rel="noopener noreferrer"}

### 입력과 출력을 한 문장으로 적기

갈색 칸 수 `brown`과 노란색 칸 수 `yellow`를 입력받아 **카펫의 가로, 세로 길이**를 배열로 반환한다.

### 예시를 손으로 풀기

`brown=10`, `yellow=2`이면 전체 칸은 12개다. `가로 × 세로 = 12`인 후보 중 `4 × 3`을 확인하면, 테두리를 제외한 안쪽은 `(4-2) × (3-2) = 2`칸이다. 노란색 칸 수와 같으므로 결과는 `[4, 3]`이다.

### 가장 단순한 방법부터 생각하기

1. 전체 칸 수 `brown + yellow`를 구한다.
2. 가능한 세로 길이를 하나씩 시도한다.
3. 전체 칸 수가 세로로 나누어떨어지면 가로를 구한다.
4. `(가로-2) × (세로-2)`가 노란색 칸 수와 같으면 답을 반환한다.

### Python 코드

```python
def solution(brown, yellow):
    total = brown + yellow

    for height in range(3, int(total ** 0.5) + 1):
        if total % height != 0:
            continue

        width = total // height

        if (width - 2) * (height - 2) == yellow:
            return [width, height]
```

### 코드가 작동하는 순서

- `total`은 갈색과 노란색을 합친 전체 칸 수다. 그러므로 `width * height == total`이어야 한다.
- 안쪽에 노란색 칸이 있으려면 세로 길이는 최소 3이다. 문제에서 가로가 세로 이상이므로 세로는 `√total`까지만 시도한다.
- `total % height != 0`이면 정수인 가로 길이를 만들 수 없다. `continue`는 현재 후보를 건너뛰고 다음 세로 길이로 넘어간다.
- 나누어떨어지면 `total // height`로 가로 길이를 구한다.
- 테두리가 사방에 한 줄씩 있으므로 안쪽 크기는 `(width-2) * (height-2)`다. 이것이 `yellow`와 같으면 정답이다.

**시간 복잡도:** 전체 칸 수를 $N$이라 하면 세로 후보를 최대 $\sqrt{N}$개 확인하므로 $O(\sqrt{N})$이다.

## 3. 순위 검색

[프로그래머스 문제 보기](https://school.programmers.co.kr/learn/courses/30/lessons/72412?language=python3){:target="_blank" rel="noopener noreferrer"}

이 문제는 앞의 두 문제보다 어렵다. 처음에는 **단순한 방법을 떠올리고, 왜 입력 크기 때문에 개선이 필요한지** 이해하는 것부터 시작한다.

### 입력과 출력을 한 문장으로 적기

지원자 정보 `info`와 검색 조건 `query`를 입력받아, **각 조건에 맞으면서 기준 점수 이상인 지원자 수**를 배열로 반환한다.

### 예시를 손으로 풀기

지원자가 `java backend junior pizza 150`이고 질문이 `java and backend and junior and pizza 100`이라면 네 조건이 모두 맞고 `150 ≥ 100`이므로 이 지원자를 한 명 센다. 조건의 `-`는 해당 항목을 검사하지 않는다는 뜻이다.

### 가장 단순한 방법부터 생각하기

질문을 하나 꺼내 모든 지원자를 검사할 수 있다. 각 지원자의 네 조건이 맞고 점수가 기준 이상이면 결과를 1 올린다. 그러나 지원자는 최대 50,000명, 질문은 최대 100,000개라서 모든 쌍을 검사하면 최대 50억 번 비교하게 된다.

따라서 **지원자를 조건별로 미리 묶고**, 묶인 점수 목록을 정렬한다. 질문이 들어오면 해당 목록에서 기준 점수 이상이 시작되는 위치를 이진 탐색한다.

### Python 코드

```python
from collections import defaultdict
from bisect import bisect_left


def solution(info, query):
    score_lists = defaultdict(list)

    for row in info:
        parts = row.split()
        conditions = parts[:4]
        score = int(parts[4])

        for mask in range(16):
            key = tuple(
                "-" if mask & (1 << i) else conditions[i]
                for i in range(4)
            )
            score_lists[key].append(score)

    for scores in score_lists.values():
        scores.sort()

    result = []

    for question in query:
        parts = question.replace(" and ", " ").split()
        key = tuple(parts[:4])
        minimum_score = int(parts[4])

        scores = score_lists[key]
        first = bisect_left(scores, minimum_score)
        result.append(len(scores) - first)

    return result
```

### 코드가 작동하는 순서

- `defaultdict(list)`는 처음 사용하는 조건 키에 빈 리스트를 자동으로 만든다.
- `row.split()`으로 지원자 정보를 다섯 부분으로 나눈다. 앞의 네 항목은 조건, 마지막 항목은 점수다.
- 네 조건 각각은 **실제 값을 요구**하거나 **`-`로 무시**할 수 있다. 따라서 지원자 한 명은 $2^4=16$개의 검색 조건에 포함된다. `mask`는 이 조합을 만들기 위한 숫자다.
- 예를 들어 `java backend junior pizza 150`은 `(java, backend, junior, pizza)`, `(java, backend, junior, -)`, `(-, -, -, -)` 등의 키에 150점을 등록한다.
- 모든 등록이 끝나면 각 조건의 점수 목록을 정렬한다.
- 질문에서 ` and `를 없애 네 조건과 최소 점수를 꺼낸다. 그 조건에 해당하는 정렬된 점수 목록을 찾는다.
- `bisect_left(scores, minimum_score)`는 최소 점수 **이상**이 처음 나타나는 위치를 반환한다. 예를 들어 `[80, 150, 210]`에서 기준이 150이면 위치 1이다. 따라서 `3 - 1 = 2`명을 결과에 넣는다.

**시간 복잡도:** 지원자 수를 $N$, 질문 수를 $Q$라 하면 전처리에서 지원자마다 16개 조건에 등록하고 정렬한다. 이후 질문마다 이진 탐색을 하므로 대략 $O(16N\log N + Q\log N)$이다. 16은 고정된 수다.

## 4. 입국심사

[프로그래머스 문제 보기](https://school.programmers.co.kr/learn/courses/30/lessons/43238?language=python3){:target="_blank" rel="noopener noreferrer"}

이 문제는 Lv.3 도전 문제다. 먼저 코드를 완성하는 것보다 **어떤 값에 이진 탐색을 적용할지** 생각해 본다.

### 입력과 출력을 한 문장으로 적기

기다리는 사람 수 `n`과 심사관별 심사 시간 `times`를 입력받아 **모든 사람이 심사를 마칠 수 있는 최소 시간**을 반환한다.

### 예시를 손으로 풀기

`n=6`, `times=[7, 10]`에서 **28분이면 가능한가?**를 확인한다.

- 첫 심사관은 `28 // 7 = 4`명 처리할 수 있다.
- 둘째 심사관은 `28 // 10 = 2`명 처리할 수 있다.
- 합쳐서 6명이므로 28분은 가능하다.

27분에는 `27 // 7 + 27 // 10 = 3 + 2 = 5`명밖에 처리하지 못하므로 예시의 최소 시간은 28분이다.

### 가장 단순한 방법부터 생각하기

1분, 2분, 3분처럼 시간을 늘려 가며 처음으로 `n`명을 처리할 수 있는 시간을 찾을 수 있다. 하지만 가능한 시간이 매우 크므로 하나씩 확인하기에는 오래 걸린다.

대신 가운데 시간 `T`를 골라 **T분 안에 n명을 심사할 수 있는가?**를 판단한다. 가능하면 더 짧은 시간에서 찾고, 불가능하면 더 긴 시간에서 찾는다. 시간이 늘면 처리할 수 있는 사람 수가 줄어들지 않으므로 이진 탐색이 가능하다.

### Python 코드

```python
def solution(n, times):
    low = 1
    high = max(times) * n

    while low <= high:
        middle = (low + high) // 2

        processed = 0
        for time in times:
            processed += middle // time

        if processed >= n:
            high = middle - 1
        else:
            low = middle + 1

    return low
```

### 코드가 작동하는 순서

- `low`와 `high`는 답이 될 수 있는 시간 범위다. 가장 느린 심사관이 혼자 모든 사람을 처리하는 `max(times) * n`분이면 확실히 충분하므로 상한으로 쓴다.
- `middle`은 **배열 인덱스가 아니라 시도해 볼 시간**이다.
- 각 심사관의 소요 시간이 `time`분이라면 `middle`분 동안 `middle // time`명을 처리할 수 있다. 이를 모두 더해 처리 가능한 인원을 구한다.
- `processed >= n`이면 현재 시간은 가능하다. 더 짧은 시간도 가능한지 보려고 `high = middle - 1`로 왼쪽 범위를 탐색한다.
- 인원이 모자라면 시간이 더 필요하므로 `low = middle + 1`로 오른쪽 범위를 탐색한다.
- 반복이 끝났을 때 `low`가 **처음으로 가능한 시간**이다.

**시간 복잡도:** 심사관 수를 $M$, 탐색 시간의 상한을 $T$라 하면 한 번의 가능 여부 검사에 $O(M)$, 이진 탐색에 $O(\log T)$가 들어 전체는 $O(M\log T)$다.

## 풀이 순서에서 기억할 점

| 문제 | 처음 떠올릴 방법 | 나중에 확인할 점 |
| --- | --- | --- |
| 모의고사 | 정답을 한 문제씩 비교 | 반복 패턴에 `%` 사용 |
| 카펫 | 가능한 가로·세로 검사 | 테두리를 뺀 내부 칸 수 확인 |
| 순위 검색 | 질문마다 모든 지원자 검사 | 입력이 커서 사전 분류와 이진 탐색 필요 |
| 입국심사 | 시간을 1분씩 늘려 검사 | 가능 여부가 한 방향으로 바뀌므로 시간에 이진 탐색 |

문제를 풀 때는 처음부터 빠른 알고리즘을 알아맞히려 하지 말고, **손으로 해 본 작업을 먼저 코드로 표현**한다. 그 방법이 느리다면 입력 크기를 보고 반복을 줄일 방법을 찾는다. 지금은 모의고사의 `enumerate`, 이중 `for`, `%`를 직접 설명하고 구현하는 것부터 시작해도 충분하다.
