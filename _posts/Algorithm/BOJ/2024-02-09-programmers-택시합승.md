---
title: "[Programmers][Python] 프로그래머스: 합승 택시 요금"
excerpt: "[Programmers][Python] 프로그래머스: 합승 택시 요금"

categories:
  - ps
tags:
  - [백준, BOJ]

toc: true
toc_sticky: true

date: 2024-02-09
last_modified_at: 2024-02-09
---

> 1일 1PS 47일차!

# 📚 문제

---

> Level 3

- Dijkstra

# 💡 풀이 과정

---

- 최소 값을 구하기 위해서는 아래와 같은 경우의 수를 고려해야한다.
  - s -> a, s - > b
  - s -> i -> a, s -> i -> b
- s -> i, i -> a, b 의 최소 비용을 구해야한다.
- 이를 각 노드를 출발지로 다익스트라 알고리즘을 사용해 최소 경로를 검색한다.
- 이후 경우의 수를 고려해 최소 경로를 구한다.

# 📌포인트

---

- 플로이드 워셜을 사용하면 더 간편하게 코드를 작성할 수 있다.
- i 노드에서 a, b 에 도착하는 것만 찾으면 된다.
  - 최소 경로 비용만 찾기에 a, b 를 출발지로 하고 다익스트라 알고리즘을 사용하면 된다.
  - i -> a 의 비용과 a -> i 비용이 동일하기 때문이다.

# 💻 코드

---

{% raw %}

```

import sys
import heapq

def solution(n, s, a, b, fares):
    # graphs = [[sys.maxsize for _ in range(n + 1)] for _ in range(n + 1)]
    # for node1, node2, cost in fares:
    #     graphs[node1][node2] = graphs[node2][node1] = cost

    graphs = {node:{} for node in range(n + 1)}
    for node1, node2, cost in fares:
        graphs[node1][node2] = graphs[node2][node1] = cost

    dp = [[]]

    for i in range(1, n + 1):
        dp.append(dijkstra(i, graphs, n))

    min_cost = dp[s][a] + dp[s][b]
    for i in range(1, n + 1):
        min_cost = min(min_cost, dp[s][i] + dp[i][a] + dp[i][b])

    return min_cost

def dijkstra(s, graphs, n):
    dp = [sys.maxsize] * (n + 1)
    dp[s] = 0

    queue = []
    heapq.heappush(queue, [dp[s], s])

    while queue:
        current_distance, current_node = heapq.heappop(queue)

        if dp[current_node] < current_distance:
            continue

        for next_node, next_distance in graphs[current_node].items():
            if current_distance + next_distance < dp[next_node]:
                dp[next_node] = current_distance + next_distance
                heapq.heappush(queue, [current_distance + next_distance, next_node])

    return dp

```

{% endraw %}
