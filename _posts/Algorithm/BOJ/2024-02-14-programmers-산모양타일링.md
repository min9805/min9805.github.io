---
title: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 산 모양 타일링"
excerpt: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 산 모양 타일링"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-14
last_modified_at: 2024-02-14
---

> 1일 1PS 52일차!

# 📚 문제

---

> Level 3

# 💡 풀이 과정

---

- 해당 문제는 점화식을 세울 수 있기에 DP 문제로 볼 수 있다. 
    - i + 1 번째를 타일로 채울 수 있는 방법은 위에 삼각형이 있냐 없냐에 따라 또 다르다.
        - 삼각형이 있는 경우
            - i 번째 경우의 수 * 삼각형 3개로 이루어진 마름모를 채울 경우의 수
            - i 번째에서 가장 오른쪽 삼각형이 없을 때의 경우의 수 * 삼각형 4개로 이루어진 정삼각형을 채울 경우의 수
            - 두 경우가 겹치는 경우는 사이의 작은 삼각형이 정삼각형 타일로 이루어진 경우
                - i 번째에서 가장 오른쪽 삼각형이 없을 때의 경우의 수 * 삼각형 3개로 이루어진 마름모를 채울 경우의 수
            - 총 경우의 수를 계산하자면
                - dp[i][0] * 3 + dp[i][1] * 4 - dp[i][1] * 3

- 삼각형이 있는 경우와 없는 경우, 가장 오른쪽 삼각형이 채워진 경우 없는 경우를 모두 계산하면 아래와 같다. 

# 📌포인트

---

- 모든 문제는 조건을 잘 읽어야한다. % 를 잘 사용해야 답이 나온다. 


# 💻 코드

---

{% raw %}

```

def solution(n, tops):
    dp = [[0, 0]  for _ in range(n)]

    if tops[0] == 1:
        dp[0] = [4, 3]
    else:
        dp[0] = [3, 2]

    for i in range(1, n):
        if tops[i] == 1:
            dp[i][0] = (dp[i -1][0] * 3 + dp[i-1][1]) % 10007
            dp[i][1] = (dp[i-1][1] + dp[i-1][0] * 2) % 10007
        else:
            dp[i][0] = (dp[i -1][0] * 2 + dp[i-1][1]) % 10007
            dp[i][1] = (dp[i-1][1] + dp[i-1][0]) % 10007

    return dp[n-1][0]
    
```

{% endraw %}
