---
title: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 주사위 고르기"
excerpt: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: 주사위 고르기"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-12
last_modified_at: 2024-02-14
---

> 1일 1PS 50일차!

# 📚 문제

---

> Level 3

# 💡 풀이 과정

---

- 모든 조합을 빠짐없이 완전 탐색하는 문제이다.
- 먼저 combination 을 사용해서 주사위를 사용할 수 있는 모든 조합을 result 에 생성한다.
    - 해당 조합은 [[[0,1], [2,3]], ... ] 이런 식으로 담겨있다.
    - 첫 번째는 1, 2 번째 주사위와 2, 3 번째 주사위를 비교한다는 뜻이다.
- 해당 result 에 있는 조합 주사위에서 나올 수 있는 합과 경우의 수를 구한다.
    - product 를 사용해 모든 조합을 구하고 sum 을 사용해 합을 구한다.
    - Counter 를 사용해서 sum 을 카운팅해 중복을 없앤다.
    - {6: 6, 7 : 10, ...} 이런 식으로 표현된다.
    - 이는 주사위의 조합에서 6이 나오는 경우는 6가지, 7이 나오는 경우는 10가지임을 나타낸다.
- 이후 조합 sub1, sub2 에 대해서 비교를 진행한다.
    - sub1에 존재하는 모든 합을 sub2의 모든 합에 비교하면서 승무패를 나타낸다.
    - sub1 = {3:10, 4:20, 5:30, ...}
    - sub2 = {2:5, 4: 15, ... }
    - 위와 같은 경우 sub1의 "3" 을 sub2 의 합들과 비교한다.
        - "2" 일 경우 sub1 이 이기는 경우이므로 이기는 경우의 수에 10 * 5 를 추가한다.
        - "4" 일 경우 sub1 이 지는 경우이므로 지는 경우의 수에 10 * 15 를 추가한다.
    - ... 이후 sub1 의 4 에 대해서도 똑같이 진행한다..
- 모든 이기고 지는 경우의 수를 저장한 table 이 완성되었으면 가장 큰 수를 뽑아낸다.
    - table[i][1] 은 비기는 수이기에 비교 대상에서 제외한다. 
    - 만약 table[i][2] 는 sub1 조합이 지는 확률이 가장 높다는 뜻이다.
        - 즉, sub2 조합이 이길 확률이 가장 높다는 뜻이므로 sub2 의 주사위 조합이 답이다.
    - 이를 통해 result 에 모든 조합을 담는 것이 아니라 중복되는 조합은 제외할 수 있다는 뜻이다.

# 📌포인트

---

- 65번째 줄에서  if max_win < t[2]: 일때 max_win = t[2] 를 t[1] 로 적어서 오답이 났다. 
    - result 를 반으로 줄이지 않았다면 시간 초과가 나는 대신 모두 정답인 것 등을 조합했을 때 디버깅을 할 수 있어야 했는데 아쉽다. 


# 💻 코드

---

{% raw %}

```

from itertools import combinations, product
from collections import  Counter


def dice_sum_prodict(d):
    dice_combinations = product(*d)
    dice_sum = [sum(c) for c in dice_combinations]
    counts = Counter(dice_sum)

    return counts


def solution(dice):
    n = len(dice)
    arr = [i for i in range(n)]
    result = []

    for i in range(1, len(arr) // 2 + 1):
        for subset1 in combinations(arr, i):
            subset2 = [x for x in arr if x not in subset1]
            if len(subset1) == len(subset2):
                result.append([list(subset1), list(subset2)])

    result = result[:int(len(result)/2)]
    print(result)

    table = []

    for r in result:
        sub1 = r[0]
        sub2 = r[1]
        sub1_dice = []
        for s1 in sub1:
            sub1_dice.append(dice[s1])

        sub2_dice = []
        for s2 in sub2:
            sub2_dice.append(dice[s2])

        sub1_all = dice_sum_prodict(sub1_dice)
        sub2_all = dice_sum_prodict(sub2_dice)


        score = [0, 0, 0]

        for number, count in sorted(sub1_all.items()):
            for number2, count2 in sorted(sub2_all.items()):
                    if number == number2:
                        score[1] += count2 * count
                    elif number > number2:
                        score[0] += count2 * count
                    else:
                        score[2] += count2 * count

        table.append(score)

    max_win = 0
    max_index = []
    for index, t in enumerate(table):
        if max_win < t[0]:
            max_win = t[0]
            max_index = [index, 0]

        if max_win < t[2]:
            max_win = t[2]
            max_index = [index, 1]

    answer = []
    for e in result[max_index[0]][max_index[1]]:
        answer.append(e + 1)

    print(table)
    return answer
    
```

{% endraw %}
