---
title: "[Programmers][Python] 프로그래머스 2018 KAKAO BLIND RECRUITMENT: 자동완성"
excerpt: "[Programmers][Python] 프로그래머스 2018 KAKAO BLIND RECRUITMENT: 자동완성"

categories:
  - ps
tags:
  - [프로그래머스, programmers]

toc: true
toc_sticky: true

date: 2024-02-18
last_modified_at: 2024-02-18
---

> 1일 1PS 56일차!

# 📚 문제

---

> Level 4

- Trie 알고리즘

# 💡 풀이 과정

---

- 문자열 prefix 에 관해서라면 Trie 알고리즘을 사용해야한다.
- words 를 Trie 알고리즘에 추가한다.
- word를 구분하기 위한 최소 글자를 추출한다.
  - 각 Char 를 타고 들어가면서 자식 노드가 1개인 경우 카운팅한다.
  - 끝날 때 카운트가 0 이라면 모두 검색해야하므로 len(string) 을 반환한다.
  - 자식 노드가 2개 이상이라면 다시 count 를 초기화한다.

# 📌포인트

---

- go, gone 이 존재할 때 둘을 구분 짓기 위해서 단어의 끝을 표현해야한다.

# 💻 코드

---

{% raw %}

```

class Trie:
    dictionary = {}

    def insert(self, string):
        current_dic = self.dictionary

        for i in string:
            if i not in current_dic:
                current_dic[i] = {}

            current_dic = current_dic[i]

        current_dic['!'] = '!'

    def prefix_len(self, string):
        current_dic = self.dictionary
        count = 0

        for i in string:
            if len(current_dic[i]) == 1:
                count += 1
            else:
                count = 0

            current_dic = current_dic[i]

        return len(string) if count == 0 else len(string) - count + 1

def solution(words):
    answer = 0
    T = Trie()

    for word in words:
        T.insert(word)

    for word in words:
        answer += T.prefix_len(word)

    return answer


```

{% endraw %}
