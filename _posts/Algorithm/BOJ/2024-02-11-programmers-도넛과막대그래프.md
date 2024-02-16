---
title: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 도넛과 막대 그래프"
excerpt: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 도넛과 막대 그래프"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-11
last_modified_at: 2024-02-14
---

> 1일 1PS 49일차!

# 📚 문제

---

> Level 2

# 💡 풀이 과정

---

- 간선들을 일단 모두 dictionary 에 저장한다.
- 만약 한 정점에서 뻗어나가는 간선의 개수가 3개 이상이라면 해당 정점이 무조건 생성된 정점이다!
    - 이는 일반적인 도넛, 막대, 8자 모양의 그래프 중에서 어떤 정점들도 3개 이상의 출발 간선을 가지지 못한다는 것이다.
    - 3개 이상의 "출발" 간선을 가지지 못함에 주의하자. "도착" 간선은 3개 이상 가질 수 있다.

- 또한 한 정점에서 아래와 같은 기준으로 그래프를 판단할 수 있다.
    - 막대 모양 그래프
        - 한 정점에서 다음 정점으로 계속해서 이동할 때 더 이상 이동할 수 없는 정점이 존재한다면 막대 모양이다.
    - 8자 모양 그래프
        - 한 정점에서 다음 정점으로 계속해서 이동 중 출발 간선의 개수가 2개가 존재한다면 해당 그래프는 8자 모양이다.
    - 도넛 모양 그래프
        - 한 정점에서 다음 정점으로 계속해서 이동 중 출발 간선의 개수가 2개인 적이 없고 출발점으로 되돌아온다면 도넛 모양이다.

- 만약 생성된 정점이 특정된다면 해당 정점에서 뻗어나가는 정점들이 어떤 유형인지만 파악하면 된다. 
    - 뻗어나가는 정점들을 위 기준에 맞춰 파악한다.
- 만약 생성된 정점이 특정되지 않는다면 "출발" 간선이 존재하는 정점들을 giver 그룹에 넣어 해당 그룹안에 정점들을 대상으로 조사한다.
    - 동일하게 해당 정점들을 기준으로 뻗어나가는 정점들에 대해 위 기준에 맞춰 파악한다.
    - 이때 방문 여부를 기록해놓는다.
    - 모든 파악이 끝났으면 방문하지 않은 노드가 전부 막대 모양의 노드라면 해당 정점이 생성된 정점이다.
        - 모든 그래프에 접촉하였고 방문하지 않은 노드는 막대 그래프의 앞쪽 노드였을 뿐이다.
        - 만약 막대 그래프가 아닌 다른 모양의 그래프라면 해당 정점에서 해당 그래프로 연결되는 출발 간선이 존재하지 않는다는 뜻이므로 생성된 정점이 아니다!


# 📌포인트

---

- 실제 시험을 보았을 때 정답처리 되었지만 이후 테스트 케이스가 추가되자 오답이 되었다.
    - 이는 방문하지 않는 노드가 막대 모양인지 파악하는 부분에서 들여쓰기가 잘못되어있어서 그런 것 같다. 

# 💻 코드

---

{% raw %}

```

def solution(edges):
    answer = [0, 0, 0, 0]
    dic = {}
    give = set()

    for edge in edges:
        if edge[0] in dic:
            dic[edge[0]].append(edge[1])
        else:
            dic[edge[0]] = [edge[1]]
        give.add(edge[1])

    giver = []
    for g in dic:
        if g not in give:
            giver.append(g)

    max_key = max(dic, key=lambda k: len(dic[k]))

    if len(dic[max_key]) >= 3:
        answer[0] = max_key
        for start in dic[max_key]:
            if start not in dic:
                answer[2] += 1
                continue
            next_node = dic[start][0]

            while True:
                if next_node not in dic:
                    answer[2] += 1
                    break
                elif len(dic[next_node]) == 2:
                    answer[3] += 1
                    break
                elif next_node == start:
                    answer[1] += 1
                    break
                next_node = dic[next_node][0]
    else:
        for g in giver:
            answer = [0, 0, 0, 0]
            visited = [False] * (len(give) + len(giver) + 1)
            visited[0] = True
            visited[g] = True

            for start in dic[g]:
                print(start)
                visited[start] = True
                if start not in dic:
                    answer[2] += 1
                    continue
                next_node = dic[start][0]
                visited[next_node] = True

                while True:
                    if next_node not in dic:
                        answer[2] += 1
                        break
                    elif len(dic[next_node]) == 2:
                        answer[3] += 1
                        break
                    elif next_node == start:
                        answer[1] += 1
                        break
                    next_node = dic[next_node][0]

            for index, tf in enumerate(visited):
                if tf == False and len(dic[index]) == 1:
                    answer[0] = g
                    return answer


    return answer
    
```

{% endraw %}
