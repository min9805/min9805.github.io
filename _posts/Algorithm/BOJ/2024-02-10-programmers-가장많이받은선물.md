---
title: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 가장 많이 받은 선물"
excerpt: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 가장 많이 받은 선물"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-10
last_modified_at: 2024-02-14
---

> 1일 1PS 48일차!

# 📚 문제

---

> Level 1

- 조건 확인만 잘하고 구현만 잘하면 되는 문제

# 💡 풀이 과정

---

- N * N gift_score[][]행렬을 생성한다.
  - gift_score[giveIndex][takeIndex] 로 주고받은 선물과 선물 지수를 표로 나타낸다. 
- total_score[] 에 총 선물 지수를 담아낸다.
- gift_score[][] 를 돌면서 주고받은 기록이 없거나 같은 수라면 선물 지수에 따라 선물을 주고 받는다.
  - 선물을 주고 받았다면 더 많은 선물을 준 사람이 선물을 받는다.

# 📌포인트

---


# 💻 코드

---

{% raw %}

```

def solution(friends, gifts):
    n = len(friends)
    gift_score = [[0 for _ in range(n)] for _ in range(n)]

    for gift in gifts:
        give, take = gift.split(" ")
        giveIndex = takeIndex = -1

        for index, f in enumerate(friends):
            if f == give:
                giveIndex = index
                break

        for index, f in enumerate(friends):
            if f == take:
                takeIndex = index
                break

        gift_score[giveIndex][takeIndex] += 1
        gift_score[takeIndex][giveIndex] -= 1

    total_score = [0] * n
    total_gift = [0] * n
    for i in range(n):
        total_score[i] = sum(gift_score[i])


    for i, fr in enumerate(gift_score):
        for index, el in enumerate(fr):
            if i == index:
                continue
            if el > 0:
                total_gift[i] += 1
            elif el == 0:
                if total_score[i] > total_score[index]:
                    total_gift[i] += 1

    print(gift_score)
    print(total_score)
    print(total_gift)
    return max(total_gift)

```

{% endraw %}
