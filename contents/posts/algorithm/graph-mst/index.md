---
title: "그래프 알고리즘이란 + 최소 신장 트리(MST)"
date: 2022-05-20
update: 2022-05-20
tags:
  - Algorithm
  - MST
  - 그래프
  - 크루스칼
  - 프림
series: "Algorithm"
---
# 🎯그래프
- 정점(V)과 간선(E)로 이루어진 자료구조이다.
- 그래프는 __사이클이 존재__ 할 수 있고 간선의 방향이 양뱡향일 수 있다.
- V개의 정점을 가지는 __무방향 그래프__ 
    - 최대 간선 갯수 = V(V-1)/2
- V개의 정점을 가지는 __방향 그래프__ 의 
    - 최대 간선 갯수 = V(V-1)

## 🧷그래프의 종류

> 
> 1. 무방향 그래프: 두 정점을 연결하는 간선에 방향이 없는 그래프이다.
>
> <img src="https://user-images.githubusercontent.com/63226023/168820953-4b4ea5a8-7471-4cd8-946f-b5948841f671.png" width="30%">
>
> 2. 방향 그래프: 두 정점을 연결하는 간선에 방향이 있는 그래프이다.
>
> <img src="https://user-images.githubusercontent.com/63226023/168822029-20a4c4a6-c0f5-40cc-bcef-2f94b55a6acd.png" width="30%">
>
> 3. 가중치 그래프: 두 정점을 이동할때 비용이 드는 그래프이다.
>
> <img src="https://user-images.githubusercontent.com/63226023/168822813-d547202b-de98-4a3b-9999-ebaab354ccc0.png" width="30%">
>
> 4. 완전 그래프: 모든 정점이 간선으로 연결되어 있는 그래프이다.
>
> <img src="https://user-images.githubusercontent.com/63226023/168822899-b5f6576f-30fa-47f3-b9a5-69c79250d875.png" width="30%">

## 그래프의 구현 방법
그래프를 구현하는 방법은 크게 __인접행렬(Adjacency Matrix)__, __인접리스트(Adjacency List)__ 방식이 있다.

두 개의 구현 방식은 각각의 장단점이 있는데 대부분 `인접리스트` 방식을 많이 사용한다.

### 인접행렬

<img src="https://user-images.githubusercontent.com/63226023/169044528-8e425dd7-6a9e-4ab4-8e89-0a4955c2d296.png">

> 인접행렬은 그래프의 노드들을 __2차원 배열__ 로 나타내어, 하나의 정점이 다른 정점과 연결이 되어있으면 `1` 아니면 `0`을 넣어서 만들어 준다.

장점
1. __두 정점의 연결 정보__ 를 알고 싶을때 2차원 배열의 위치가 곧 연결 정보이기 때문에 O(1)의 시간복잡도로 알 수 있다.
2. 구현이 비교적 간단하다.

단점 
1. 2차원 배열에 모든 정점에 대한 간선 정보를 넣어줘야 하므로 O(n²)의 시간복잡도가 소요된다.
2. 2차원 배열을 사용하므로 필요 이상의 공간을 사용할 가능성이 있다.

### 인접리스트

<img src="https://user-images.githubusercontent.com/63226023/169202076-00adc01a-00c7-4123-9bf5-7e7cc35bb676.png" width="50%">

> 인접리스트는 그래프의 노드간의 연결을 __리스트__ 로 나타내어, 각 1,2,3,4,5의 정점에 대한 리스트 배열을 만들어 관계를 설정해준다.

장점
1. __두 정점의 연결 정보__ 를 탐색할때 O(n) 시간복잡도로 알 수 있다.
2. 필요한 연결정보만 리스트로 설정하여 사용하므로 공간 낭비가 적다.

단점
1. 특정 두 정점에 대한 열결 정보를 알고 싶다면 인접행렬에 비하여 느리다.

# 최소 스패닝 트리
> 🌳 최소 신장 트리(Minimum Spanning Tree)란
> 
>   - 그래프의 __최소 연결 그래프__ -> __Spanning Tree__
>   - __Spanning Tree__ 에서 간선(E)의 __가중치의 합이 최소__ 가 되는 트리 -> __Minimum Spanning Tree__

## 크루스칼
크루스칼 알고리즘은 __가중치가 있는무향 연결 그래프__ 가 주어질 때, 최소 스패닝 트리(MST)를 구하는 알고리즘이다. 

- __모든 간선__ 에 대하여 가중치가 적은 순서대로 정렬하여 __그리디 하게__ 뽑는다.
- 가중치가 적은 순서대로 뽑아 두 정점이 `연결`되어 있지 않다면 두 정점을 `union`하여 연결한다.
- 두 정점이 `연결`되었다는 것은 하나의 집합을 이루었다고 할 수 있다.
- 최소 신장 트리를 구하기 위해서 같은 집합(사이클이 형성 되는지)에 속하는지에 대한 판별을 하는 과정이 필요하다.

### Union & Find(합집합 찾기)
사이클의 형성에 대한 판단으로 `Union & Find` 다른 말로 __서로소 집합(Disjoint-Set)__ 알고리즘이라고 한다.

서로 다른 두 집합을 합치는 Union 연산, 알고 싶은 원소가 어느 집합에 포함되는지 찾는 Find 연산이라는 의미로 이름이 붙여졌다.

아래의 예시는 Union & Find를 통하여 사이클의 형성을 알아보는 과정을 보여준다.

> <img src="https://user-images.githubusercontent.com/63226023/169308331-f4c9356f-693b-41eb-b099-c4b95baf6fdc.png">
>
> - 동작 과정
> 
> 1. 모든 정점의 개수의 크기의 배열을 생성하여 자기자신으로 초기화 한다.
> 
> 2. 두 노드간의 루트 노드(부모 노드)가 다르다면 -> 서로소 집합 -> Union 연산으로 두 노드를 연결해주고, 부모 노드를 바뀌준다. 같다면 사이클이 형성되므로 연결 불가능

부모 노드를 찾는 Find 연산은 다음과 같다.

```kotlin
// 재귀적으로 쭉 찾기
fun getParentNode(node: Int): Int {
    return if (parentNode[node] == node) {
        node
    } else { // parent 노드가 자기자신이 아님 -> 연결된 부모가 있음
        getParentNode(parentNode[node])
    }
}

// 경로 압축
fun getParentNode(node: Int): Int {
    if (parentNode[node] != node) {
        // 바로 갱신하는 동시에 재귀 호출
        parentNode[node] = getParentNode(parentNode[node])
    }
    return parentNode[node]
}
```
 
## 프림
크루스칼과 마친가지로 최소 스패닝 트리(MST)를 구하는 알고리즘이다. 프림 알고리즘은 크루스칼과 다르게 임의의 정점을 시작점으로 잡는다. 

- __시작 정점__ 을 기준으로 가중치가 가장 작은 간선과 __연결된 정점__ 을 선택하여 트리를 확장해 가는 방식이다.
- 각 정점들은 인접한 정점 중 __최소 비용 간선__ 인 정점을 선택하여 큐에 추가한다.
- __우선순위 큐__ 를 이용하여 임의의 정점에서 인접한 정점의 __간선의 가중치를 기준으로 정렬__ 한다.
- 가중치가 최소인 정점을 연결후 다시 인접한 정점을 넣고 정렬한다.

> <img src="https://user-images.githubusercontent.com/63226023/169350972-12a2a1ff-416c-4b9e-a779-b62f757f03ee.png">
>
> - 동작 과정
> 
> 1. 임의의 정점에서 연결된 인접된 노드중 __가중치가 최소인 간선__ 을 선택한다.
> 
> 2. 크루스칼과 마찬가지로 사이클 형성여부는 Union & Find 연산으로 확인한다.
>
> 3. 연결이 되었다면 선택된 간선에 연결된 정점 중에 __가중치가 최소인 간선__ 을 선택한다.(방문하지 않은 노드중)

- - -

# References

- [https://ko.wikipedia.org/wiki/%ED%94%84%EB%A6%BC_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A6%BC_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)
- [https://velog.io/@yuhyerin/%EA%B7%B8%EB%9E%98%ED%94%84%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98](https://velog.io/@yuhyerin/%EA%B7%B8%EB%9E%98%ED%94%84%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)