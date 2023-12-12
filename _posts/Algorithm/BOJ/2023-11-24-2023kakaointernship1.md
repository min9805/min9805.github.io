---
title: "[프로그래머스][Python] 2022 KAKAO TECH INTERNSHIP : 두 큐 합 같게 만들기"
excerpt: "[프로그래머스][Python] 2022 KAKAO TECH INTERNSHIP : 두 큐 합 같게 만들기"

categories:
  - ps
tags:
  - [프로그래머스, 카카오]

toc: true
toc_sticky: true

date: 2023-11-24
last_modified_at: 2023-12-05
---

> 1일 1PS 35일차!

# 📚 문제

---

> Lv 1

- MBTI 처럼 각 유형에 대한 질문이 정해져있고 질문에 대한 답으로 유형에 대해서 +- 계산해 판단할 수 있다.
- 예를 들어 AN 에서 1을 골랐다면 N 형에 3점을 주고 NA 에서 1번을 골랐다면 A 형에 2점을 주는 것을 (+3) + (-2) 로 AN 의 유형이 양수이므로 N 형으로 판단하는 것이다.

# 💡 포인트

---

1. survey 가 어떤 유형에 속하는 지, 또한 정방향 유형인지 판단한다.
2. 유형값을 저장하는 ch 배열에 응답값들을 더한다.
3. 총 합이 양수인지 음수인지 판단해 유형값을 출력한다.

# 💻 코드

---

{% raw %}

```

def solution(survey, choices):
    answer = ""
    # RT, CF, JM, AN
    form = ['RT', 'CF', 'JM', 'AN']
    ch = [0] * 4

    for index, s in enumerate(survey):
        c = choices[index] - 4
        if c == 0:
            continue
        if s in form:
            for i, f in enumerate(form):
                if s == f:
                    ch[i] += c
                    break
        else:
            for i, f in enumerate(form):
                if s[0] in f:
                    ch[i] -= c

    for i in range(4):
        if ch[i] > 0:
            answer += form[i][1]
        else:
            answer += form[i][0]

    return answer


```

{% endraw %}
