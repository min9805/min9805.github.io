---
title: "[Programmers][Python] 프로그래머스: 연속된 부분 수열의 합"
excerpt: "[Programmers][Python] 프로그래머스: 연속된 부분 수열의 합"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-17
last_modified_at: 2024-02-17
---

> 1일 1PS 55일차!

# 📚 문제

---

> Level 2


# 💡 풀이 과정

---

- 연속된 합 계산 문제는 익숙하고 쉽다.
- [0:1] 배열에서 시작해서 target 값 보다 작으면 오른쪽으로 한칸 늘려 [0:2] 로 합을 구하고
- 합이 target 값 보다 크다면 왼쪽을 한칸 줄여서 [1:2] 로 합을 구한다. 

# 📌포인트

---

- While 문을 True 로 했지만 end <= len(sequence) 로 변경하면 될 것 같다. 
  - 실제 break 문을 없애고 
  -     while end < len(sequence) or total > k:
  - 해당 조건도 정답으로 나온다.
  - end가 끝까지 도착하더라도 target 보다 큰 경우 왼쪽을 줄여나가는 게 필요하다.
  
# 💻 코드

---

{% raw %}

```

def solution(sequence, k):
    answer = []

    start = 0
    end = 1
    total = sequence[start]
    min_length = len(sequence)

    while True:
        if total < k:
            if end == len(sequence):
                break
            total += sequence[end]
            end += 1

        elif total > k:
            total -= sequence[start]
            start += 1

        if total == k:
            if min_length > end - 1 - start:
                min_length = end - 1 - start
                answer = [start, end - 1]
            total -= sequence[start]
            start += 1


    return answer

```

{% endraw %}
