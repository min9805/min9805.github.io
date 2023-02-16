---
title: "[Algorithm] Segment Tree"
excerpt: "세그먼트 트리의 시간복잡도와 구현"

categories:
  - algorithm
tags:
  - [Algorithm, SegmentTree]

toc: true
toc_sticky: true

date: 2023-02-16
last_modified_at: 2023-02-16
---

# 세그먼트 트리

세그먼트 트리는 point update와 range query를 모두 O(logN)의 시간에 처리할 수 있는 시간/공간 복잡도가 매우 효율적인 자료구조이다.

가장 기초적인 형태의 세그먼트 트리에 대해서 알아보도록 하겠다. 가장 기초적이라 함은 교환법칙이 성립하는 연산에 대한 range query를 처리하는 세그먼트 트리를 말한다. 예를 들어, 구간 최소값이나 구간 합은 min(min(a, b), c) = min(a, min(b, c))이고, (a + b) + c = a + (b + c)이므로 min과 +연산은 둘 다 교환법칙이 성립한다.

세그먼트 트리는 이렇게 교환법칙이 성립하는 연산의 구간 쿼리를 처리할 수 있으며, 아래 수열에서의 구간 합을 예시로 든다.

| a0  | a1  | a2  | a3  | a4  |
| --- | --- | --- | --- | --- |
| 5   | -1  | 3   | 2   | 8   |

## 완전 탐색과 DP

```
int a[5] = {5, -1, 3, 2, -8};

// a[i]를 x로 바꿈
void set(size_t i, int x) {
           a[i] = x;
}

// [ l , r ) 구간의 합
int query(size_t l, size_t r) {
           int result = 0;
           for (size_t i = l; i < r; ++i) {
                     result += a[i];
           }
           return result;
}
```

구간 합 쿼리가 있을 때마다 모든 값을 더하지만 수열의 값 하나를 바꾸는 건 바로 할 수 있다.

> point update O(1), range query O(N)이다.

DP 를 사용해 미리 O(N^2)개의 구간 합을 미리 구해 놓는다면, 쿼리를 O(1)에 구할 수 있다.

| dp    | r = 0 | r = 1 | r = 2 | r = 3 | r = 4 | r = 5 |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| l = 0 | 0     | 5     | 4     | 7     | 9     | 1     |
| l = 1 | /     | 0     | -1    | 2     | 4     | -4    |
| l = 2 | /     | /     | 0     | 3     | 5     | -3    |
| l = 3 | /     | /     | /     | 0     | 2     | -6    |
| l = 4 | /     | /     | /     | /     | 0     | -8    |

> dp[l][r] = l이상 r미만 범위의 구간 합

하지만 수열의 값 하나가 바뀐다면 그 값을 포함하는 모든 구간 합도 바뀌어야 하므로 위의 DP 테이블에서 O(N^2)개의 값을 갱신해줘야 한다.

| a0  | a1  | a2  | a3                                  | a4  |
| --- | --- | --- | ----------------------------------- | --- |
| 5   | -1  | 3   | <span style="color:red"> 100 <span> | 8   |

| dp    | r = 0 | r = 1 | r = 2 | r = 3 | r = 4                               | r = 5                              |
| ----- | ----- | ----- | ----- | ----- | ----------------------------------- | ---------------------------------- |
| l = 0 | 0     | 5     | 4     | 7     | <span style="color:red"> 101 <span> | <span style="color:red"> 99 <span> |
| l = 1 | /     | 0     | -1    | 2     | <span style="color:red"> 102 <span> | <span style="color:red"> 94 <span> |
| l = 2 | /     | /     | 0     | 3     | <span style="color:red"> 103 <span> | <span style="color:red"> 95 <span> |
| l = 3 | /     | /     | /     | 0     | <span style="color:red"> 100 <span> | <span style="color:red"> 92 <span> |
| l = 4 | /     | /     | /     | /     | 0                                   | -8                                 |

> point update O(N^2), range query O(1)이다.

위의 두 방식에서 알 수 있듯이, 미리 여러 개의 구간 합을 구해 둘 경우 쿼리는 빨라지지만 업데이트는 느려진다.

세그먼트 트리는 구간을 똑똑하게 나눠서 어떤 구간이 주어지든 O(logN)개의 합으로 나타낼 수 있으며, 어떤 값이 바뀌어도 같이 변해야 하는 구간은 O(logN)개이다.

## 세그먼트 트리

세그먼트 트리는 아래처럼 생긴 이진 트리입니다. 구간의 길이가 1인(원소의 개수가 하나인) 구간은 리프 노드가 되고, 나머지 노드는 자식 노드를 합친 구간 합을 들고 있다.

배열의 길이가 N이라면 세그먼트 트리의 노드 개수는 2N - 1개가 된다.

| a0  | a1  | a2  | a3  | a4  |
| --- | --- | --- | --- | --- |
| 5   | -1  | 3   | 2   | 8   |

<img width="500" alt="image" src="https://user-images.githubusercontent.com/56664567/219282387-cfa9a308-aa37-4a7a-84b2-1e266556fdff.png">

힙과 마찬가지로 세그먼트 트리는 배열로 구현할 수 있다. 0번 노드는 버리고, 1번 노드가 루트가 되는 방식으로 구현하면 완전 이진 트리의 모습을 하며, 필요한 배열의 길이는 2N이 되고, 리프 노드의 인덱스는 [ N , 2N )이다.

완전 이진 트리이기 때문에 높이는 O(logN)이 된다.

```
constexpr size_t n = 5;

int a[n] = {5, -1, 3, 2, -8};
int segtree[2 * n];

void init() {
           for (size_t i = 0; i < n; ++i) {
                     segtree[i + n] = a[i];
           }
           for (size_t i = n - 1; i != 0; --i) {
                     segtree[i] = segtree[i << 1] + segtree[i << 1 | 1];
           }
}
```

## Point Update

원소 하나(리프 노드)가 바뀌면 그 원소를 포함하는 모든 구간(리프 노드의 모든 조상)의 값도 바뀌어야 한다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/219282716-7685035d-c480-4d54-94c7-1bd465524509.png">

```
// i번째 값을 x로 바꿈

void update(size_t i, int x) {
           segtree[i += n] = x;
           while (i >>= 1) {
                     segtree[i] = segtree[i << 1] + segtree[i << 1 | 1];
           }
}
```

i번째 리프 노드의 인덱스는 i + N이다. 그 값을 바꾸고자 하는 값으로 바꿔주고, 해당 리프 노드부터 루트 노드까지 부모 노드를 타고 올라가면서 조상 노드의 값을 갱신해준다.

## Range Query

구간 쿼리가 들어오면, 몇 개의 노드를 골라서 해당 구간을 정확히 덮어야 한다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/219283160-7cb62ad2-f9d4-424e-9a57-3c4f0c0638ba.png">

구간의 시작점 노드 l과 구간의 끝점의 다음 노드 r에서 부모 노드로 올라가면서, 해당 노드에 적힌 값을 결과값에 더해야 하면 더하고, 아니라면 건너뛰는 작업을 반복한다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/219283507-b941e21a-c634-44da-b50f-568fc1710c27.png">

예를 들어, l노드가 왼쪽 그림처럼 부모 노드의 왼쪽 자식일 경우(인덱스가 짝수일 경우), 굳이 저기 적힌 값을 더할 필요가 없다. 부모 노드 p에 적힌 값을 더하면 l 뿐만 아니라 오른쪽 자식의 값도 한 번에 더할 수 있으므로 건너뛴다.

하지만 오른쪽 그림처럼 부모 노드의 오른쪽 자식일 경우(인덱스가 홀수일 경우), 지금 더하지 않고 부모 노드로 올라가게 되면 구간에 포함되지 않는 왼쪽 자식의 값이 섞이게 된다. 따라서 l에 적힌 값을 더하고 구간에서 l 노드를 제외하면 된다.

r 노드도 같은 논리로 부모 노드로 계속 올라가면서 필요할 때만 값을 더해주면 된다.

```
// [ l , r ) 구간 합

int query(size_t l, size_t r) {
           int result = 0;
           for (l += n, r += n; l != r; l >>= 1, r >>= 1) {
                     if (l & 1) result += segtree[l++];
                     if (r & 1) result += segtree[--r];
           }
           return result;
}
```

---

위에서 만든 세그먼트 트리는 노드 순서가 뒤죽박죽이다. 왼쪽 자식이 오른쪽 구간을, 오른쪽 자식이 왼쪽 구간을 나타내는 경우도 있고 중간이 끊긴 구간을 나타내는 노드도 있다. 교환법칙이 성립하지 않는 일반적인 연산에 대한 쿼리를 처리하기 위해선 노드가 나타내는 구간이 정렬 상태를 유지해야 한다.

이를 위해서 비재귀적으로 구현한 세그먼트 트리는 아래처럼 포화 이진 트리 모양이다. 포화 이진 트리로 만들기 위해 리프 노드의 개수가 2의 거듭제곱이 되어야 하므로 더미 노드를 둔다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/219283930-e5a632b6-9dc0-4723-a54b-418fdaa3ab68.png">

또는 재귀적 방식으로 트리를 만들어서 노드 순서를 조정할 수도 있다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/56664567/219283955-4a6327cf-883f-41ec-8de5-94053e19e4dd.png">
