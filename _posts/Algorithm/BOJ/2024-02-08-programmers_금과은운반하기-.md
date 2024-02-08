---
title: "[Programmers][Python] 프로그래머스: 금과 은 운반하기"
excerpt: "[Programmers][Python] 프로그래머스: 금과 은 운반하기"

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

- 이분탐색
- [https://min9805.github.io/ps/boj1561/] 해당 문제와 비슷하기에 이분 탐색임을 빠르게 알 수 있었다.

# 💡 풀이 과정

---

- 시간을 제시하고 해당 시간 안에 금과 은을 모두 전달할 수 있는 지 확인한다.
- end 시간을 제시하기 위해서 각 도시에서 광물을 모두 전달할 때까지 걸리는 최대 시간을 금과 은 별로 구해 더해주었다.
- mid 시간에 금과 은을 모두 전달 가능한 지 살펴보아야 한다.
  - 해당 시간에서 전달 가능한 최대, 최소의 금과 은의 양을 계산한다.
  - a, b 를 최소 금, 은의 양으로 뺴주고 이 중 max 값이 광물의 최대 최소 차보다 작고 (쉽게 말해 a, b 가 최소 금, 은 보다 크고) 부족량의 합이 최대 최소 차보다 작을 경우에 시간이 남는 경우이므로 end = mid
  - a, b 가 최대 금, 은보다 작고 부족량을 채울 수 없을 때 시간이 부족하므로 start = mid + 1 을 진행한다.

# 📌포인트

---

- [x + y for x, y in fx, fy]
  - 리스트 간의 연산 및 비교 등을 위와 같이 진행할 수 있다.
- [ x / 2 for x in fx]
  - 같은 연산을 리스트에 반복할 때 위 처럼 간단하게 나타낼 수 있다.
- 이분탐색
  - 역시 mid == target 는 진정한 이분 탐색이 아님을 강조한다.

# 💻 코드

---

{% raw %}

```

def solution(a, b, g, s, w, t):
    start = 1
    end = max(time * (move * 2 + 1) for time, move in zip(t, [gold // weight for gold, weight in zip(g, w)])) + max(time * (move * 2 + 1) for time, move in zip(t, [silver // weight for silver, weight in zip(s, w)]))

    while start < end:
        mid = (start + end) // 2
        move = [((mid // time) + 1) // 2 for time in t]
        move_weight = [ weight * m for weight, m in zip(w, move)]

        min_gold, max_gold, min_silver, max_silver = 0, 0, 0, 0
        for i in range(len(move)):
            if move_weight[i] >= s[i] + g[i]:
                min_gold += g[i]
                max_gold += g[i]
                min_silver += s[i]
                max_silver += s[i]
            else:
                if move_weight[i] <= g[i]:
                    max_gold += move_weight[i]
                elif move_weight[i] > g[i]:
                    max_gold += g[i]
                    min_silver += (move_weight[i] - g[i])

                if move_weight[i] <= s[i]:
                    max_silver += move_weight[i]
                elif move_weight[i] > s[i]:
                    max_silver += s[i]
                    min_gold += (move_weight[i] - s[i])

        dif = max_gold - min_gold
        dif_a = a - min_gold
        dif_b = b - min_silver

        if max(dif_a, dif_b) <= dif and dif >= dif_a + dif_b:
            end = mid
        else:
            start = mid + 1

    return end

```

{% endraw %}
