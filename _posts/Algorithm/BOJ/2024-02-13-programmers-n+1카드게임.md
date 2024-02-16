---
title: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: n + 1 카드게임"
excerpt: "[Programmers][Python] 프로그래머스 2024 KAKAO WINTER INTERNSHIP: n + 1 카드게임"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-13
last_modified_at: 2024-02-14
---

> 1일 1PS 51일차!

# 📚 문제

---

> Level 3

# 💡 풀이 과정

---

- ...

# 📌포인트

---

- ...


# 💻 코드

---

{% raw %}

```

def solution(coin, cards):
    n = len(cards)
    hands = cards[:int(n / 3)]
    card_index = int(n / 3)

    need_cards = []
    pair = 0

    for card1 in hands:
        if n + 1 - card1 not in hands:
            need_cards.append(n + 1 - card1)
        else:
            pair += 1

    pair = pair // 2

    sub_cards = [card for card in cards[card_index: card_index + 2 * (pair + 1)]]
    card_index += 2 * (pair + 1)
    round = 1 + pair
    pair = 0

    while card_index < len(cards):
        if coin > 0:
            removeList = []
            for card1 in need_cards:
                if card1 in sub_cards:
                    removeList.append(card1)
                    sub_cards.remove(card1)
                    coin -= 1
                    pair += 1
                    if coin <= 0:
                        break
            for rm in removeList:
                need_cards.append(rm)

        if pair <= 0:
            if coin >= 2:
                for card2 in sub_cards:
                    if (n + 1 - card2) in sub_cards:
                        sub_cards.remove(card2)
                        sub_cards.remove(n + 1 - card2)
                        coin -= 2
                        pair += 1
                        if coin <= 0:
                            break

        if pair <= 0:
            break

        pair -= 1
        round += 1
        sub_cards.append(cards[card_index])
        sub_cards.append(cards[card_index + 1])
        card_index += 2

    max_round = (n // 3) + 1
    if card_index > len(cards):
        return max_round
    else:
        return round
    
```

{% endraw %}
