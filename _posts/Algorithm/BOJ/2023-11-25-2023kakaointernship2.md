---
title: "[프로그래머스][Python] 2022 KAKAO TECH INTERNSHIP : 성격 유형 검사하기"
excerpt: "[프로그래머스][Python] 2022 KAKAO TECH INTERNSHIP : 성격 유형 검사하기"

categories:
  - ps
tags:
  - [프로그래머스, 카카오]

toc: true
toc_sticky: true

date: 2023-11-25
last_modified_at: 2023-12-05
---

> 1일 1PS 36일차!

# 📚 문제

---

> Lv 2

- 연속된 부분 수열의 합을 찾아가는 과정과 동일하다.
- 목표값을 계산하고 하나의 큐를 선택한다.
- 큐의 총 합이 목표값보다 작으면 push 하고 목표값보다 크면 deque 한다
- 큐의 길이가 <= 300,000 이므로 완전탐색을 진행한다. 


# 💡 풀이 과정

---

1. 총 합의 절반을 목표값으로 정한다. 
2. q1 의 총 합이 목표값보다 작으면 append 하고 목표값보다 크면 popleft 한다.


# 💻 코드

---

{% raw %}

```

from collections import deque

def solution(queue1, queue2):
    q1 = deque(queue1)
    q2 = deque(queue2)

    sum = 0
    for q in queue1:
        sum += q
    q1_sum = sum
    for q in queue2:
        sum += q

    sum = int(sum/2)

    cnt = 0
    for _ in range((len(queue1) + len(queue2)) * 2):
        if q1_sum > sum:
            if not q1:
                return -1
            temp1 = q1.popleft()
            q2.append(temp1)
            q1_sum -= temp1
            cnt += 1
        elif q1_sum < sum:
            if not q2:
                return -1
            temp2 = q2.popleft()
            q1.append(temp2)
            q1_sum += temp2
            cnt += 1

        if q1_sum == sum:
            return cnt

    return -1

```

{% endraw %}
