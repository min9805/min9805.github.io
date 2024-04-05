---
title: "[코딩테스트] 코딩 테스트 전 확인 리스트"
excerpt: "코딩 테스트 전 확인 리스트"

categories:
  - algorithm
tags:
  - [Algorithm, SegmentTree]

toc: true
toc_sticky: true

date: 2024-04-05
last_modified_at: 2024-04-05
---

# Python

## 라이브러리 사용법

### Queue
```
from collections import deque

queue = deque()
queue.append()
queue.popleft()
queue[0]
```
### Dictionary
```
dict = {"example" : 0, "test" : 1}
dict["example"]
```
### Sort
```
answer.sort(key=lambda x: x[dict["example"]])
```
# Java

## 라이브러리 사용법

### Queue
```
import java.util.Queue;

static Queue<int[]> queue = new LinkedList<>();
queue.add()
queue.poll()
queue.peek()
```
### Dictionary
```
import java.util.HashMap;
HashMap<String, Integer> dict = new HashMap<>(){ put("example", 0); put("test", 1); };
dict.get("example")
```
### Sort
```
answer.sort((a, b) -> Integer.compare(a[dict.get("example")], b[dict.get("example")]));
```
```
import java.util.Comparator;

List<int[]> answer = new ArrayList<>();

Comparator<int[]> customComparator = new Comparator<>() {
    @Override
    public int compare(int[] a, int[] b) {
        return Integer.compare(a[dict.get(sort_by)], b[dict.get(sort_by)]);
    }
};

answer.sort(customComparator);
```
```
import java.util.Comparator;

int[][] answer = new int[list.size()][4];

Arrays.sort(answer, new Comparator<int[]>() {
    @Override
    public int compare(int[] s1, int[] s2) {
        return s1[dict.get(sort_by)] - s2[dict.get(sort_by)];
    }
});
```

## 주의할 점 

- String 비교 시 equals() 사용
  - String 은 객체이므로 equals 를 사용해야한다.
  - 직접 선언할 시 주솟값이 같지만 그래도 꼭 equals 를 사용하자.