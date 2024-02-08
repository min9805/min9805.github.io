---
title: "[Programmers][Python] 프로그래머스: 정수 삼각형"
excerpt: "[Programmers][Python] 프로그래머스: 정수 삼각형"

categories:
  - ps
tags:
  - [백준, BOJ]

toc: true
toc_sticky: true

date: 2024-02-08
last_modified_at: 2024-02-08
---

> 1일 1PS 46일차!

# 📚 문제

---

> Level 3

- DFS?
- DP

- 레벨 3 치고는 너무 쉬운 문제

# 💡 풀이 과정

---

- 처음에는 DFS 인가 생각했다.
  - 근데 층을 내려갈 수록 2\*\*i 개의 분기가 발생하고 높이가 최대 500 이므로 불가능한 방법이다.
- 작은 삼각형을 하나씩 보자면 결국 자신의 부모 노드 중 큰 것을 사용해야 하기에 DP 가 가능하다.

# 📌포인트

---

# 💻 코드

---

{% raw %}

```

def solution(triangle):
    height = len(triangle)
    dp = [[0] * i for i in range(1, height + 1)]
    dp[0][0] = triangle[0][0]

    for i in range(1, height):
        for j in range(len(dp[i])):
            if j == 0:
                dp[i][j] = dp[i-1][j] + triangle[i][j]
            elif j == len(dp[i]) - 1:
                dp[i][j] = dp[i-1][j-1] + triangle[i][j]
            else:
                dp[i][j] = max(dp[i-1][j-1], dp[i-1][j]) + triangle[i][j]

    return max(dp[height - 1])

```

{% endraw %}
