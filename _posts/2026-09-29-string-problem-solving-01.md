---
title: "문자열 문제 풀이 모음 01"
date: 2026-09-29 00:00:00 +0900
categories: [Study, Algorithm]
tags: [programmers, python, algorithm, string, sorting]
description: 프로그래머스 문자열 정렬, 비밀지도, 접미사 배열, 문자열 묶기 문제 풀이
math: true
---

## 문자열 내 마음대로 정렬하기

> **문제 요약**
> 문자열 배열과 인덱스 `n`이 주어지면 각 문자열의 n번째 문자를 기준으로 오름차순 정렬하는 문제이다. n번째 문자가 같다면 문자열 전체의 사전순으로 순서를 정한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/12915)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/12915?language=python3)

1. 입력: 문자열로 구성된 리스트, 인덱스 n번째
2. 출력: 인덱스 n번째 글자를 기준으로 오름차순 정렬한 리스트
3. 내가 손으로 풀면:
   - 각 문자열의 n번째 문자를 확인한다.
   - n번째 문자가 더 앞선 문자열이 앞으로 오게 정렬한다.
   - n번째 문자가 같다면 문자열 전체를 비교한다.
4. 반복해야 하는 것: 리스트에 있는 문자열들을 정렬하면서 비교
5. 조건:
   - `A[n] < B[n]` → A가 앞
   - `A[n] > B[n]` → B가 앞
   - `A[n] = B[n]` → 문자열 전체를 사전순 비교
6. 기억해야 하는 값: 없음
7. 예상 시간복잡도: $O(n \log n)$ → 일반적인 비교 기반 정렬

두 개씩 비교: $8 \rightarrow 4 \rightarrow 2 \rightarrow 1 = \log_2 n$, 전체 n개 처리 ⇒ $n \times \log n$

```text
answer에 첫 문자열 넣기

나머지 strings를 하나씩 꺼냄:
    inserted = False

    answer를 앞에서부터 확인:
        새 문자열이 현재 문자열보다 앞이면:
            그 위치에 insert
            inserted = True
            break

    끝까지 확인했는데 inserted가 False면:
        맨 뒤에 append
```

```python
def solution(strings, n):
    answer = []
    answer.append(strings[0])

    for str1 in strings[1:]:
        inserted = False

        for index, str2 in enumerate(answer):
            if str1[n] < str2[n]:
                answer.insert(index, str1)
                inserted = True
                break
            elif str1[n] == str2[n]:
                if str1 < str2:
                    answer.insert(index, str1)
                    inserted = True
                    break

        if not inserted:
            answer.append(str1)

    return answer
```

### 짧은 버전

```python
def solution(strings, n):
    return sorted(strings, key=lambda x: (x[n], x))
```

## [1차] 비밀지도

> **문제 요약**
> 숫자로 암호화된 두 장의 지도를 겹쳐 원래의 비밀지도를 복원하는 문제이다. 각 숫자를 n자리 이진수로 변환하고, 두 지도 중 하나라도 벽인 칸은 `#`, 두 지도 모두 공백인 칸은 공백으로 표시한다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/17681)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/17681)

1. 입력: 지도의 한 변 크기 n, 2개의 정수 배열 arr1, arr2
2. 출력: `#`과 공백으로 구성된 문자열 배열
3. 내가 손으로 풀면:
   - 각 정수 배열의 같은 위치 원소를 이진수로 본다.
   - 두 원소를 OR 연산한다.
   - 결과의 각 비트를 확인한다.
   - 1이면 `#`, 0이면 공백으로 바꾼다.
   - 완성된 문자열을 answer에 넣는다.
4. 반복해야 하는 것:
   - arr1과 arr2의 같은 인덱스 원소들을 하나씩 처리
   - OR 결과의 각 비트 확인
5. 조건:
   - `arr1[i] | arr2[i]`
   - 0 → 공백
   - 1 → `#`
6. 기억해야 하는 값: 해독한 비밀지도를 저장할 answer
7. 예상 시간복잡도: $O(n^2)$
   - n줄마다 × 줄마다 n칸 → $n^2$

```python
def solution(n, arr1, arr2):
    answer = []

    for i, j in zip(arr1, arr2):
        record = f"{i | j:0{n}b}"
        record = record.replace("1", "#").replace("0", " ")
        answer.append(record)

    return answer
```

- `replace()`: 원본을 바꾸지 않음
- f-string: `f"{value:0{size}b}"` → 자릿수를 size로 고정하고 이진수 문자열로 변환
- 짧은 풀이: zip + 비트 OR + f-string + replace + 리스트 컴프리헨션

## 접미사 배열

> **문제 요약**
> 문자열의 각 인덱스에서 시작해 끝까지 이어지는 모든 접미사를 만든 뒤, 이를 사전순으로 정렬하여 반환하는 문제이다. 예를 들어 `banana`의 접미사에는 `banana`, `anana`, `nana`, `ana`, `na`, `a`가 있다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/181909)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/181909)

1. 입력: 문자열 my_string
2. 출력: 모든 접미사를 사전순으로 정렬한 문자열 배열
3. 내가 손으로 풀면:
   - 문자열의 각 인덱스를 시작점으로 해서 끝까지 자른다.
   - 자른 접미사들을 배열에 저장한다.
   - 배열을 사전순으로 정렬한다.
4. 반복해야 하는 것: 문자열의 각 시작 인덱스
5. 조건: 없음
6. 기억해야 하는 값: 자른 접미사
7. 예상 시간복잡도: 접미사 n개 생성 + 정렬 → $O(n \log n)$

```python
def solution(my_string):
    answer = []

    for i in range(len(my_string)):
        answer.append(my_string[i:])

    answer.sort()

    return answer
```

### 짧은 코드

```python
def solution(my_string):
    return sorted([my_string[i:] for i in range(len(my_string))])
```

슬라이싱 + range + 리스트 컴프리헨션 + sorted

## 문자열 묶기

> **문제 요약**
> 문자열 배열의 원소를 길이가 같은 문자열끼리 묶었을 때 가장 많은 문자열이 포함된 그룹의 크기를 구하는 문제이다. 문자열의 내용이 아닌 길이별 등장 횟수를 세는 것이 핵심이다. [공식 문제](https://school.programmers.co.kr/learn/courses/30/lessons/181855)

[문제 바로가기](https://school.programmers.co.kr/learn/courses/30/lessons/181855)

1. 입력: 문자열 배열 strArr
2. 출력: 같은 길이의 문자열끼리 묶었을 때 가장 많이 모인 그룹의 개수
3. 내가 손으로 풀면:
   - 문자열을 하나씩 본다.
   - 각 문자열의 길이를 확인한다.
   - 해당 길이가 몇 번 나왔는지 센다.
   - 가장 많이 나온 개수를 반환한다.
4. 반복해야 하는 것: strArr의 문자열을 하나씩 확인
5. 조건: 특별한 조건문보다는 "문자열 길이별 개수 세기"
6. 기억해야 하는 값: 각 문자열 길이가 몇 번 등장했는지
7. 예상 시간복잡도: $O(N)$

```python
def solution(strArr):
    count = {}

    for s in strArr:
        length = len(s)

        if length in count:
            count[length] += 1
        else:
            count[length] = 1

    return max(count.values())
```
