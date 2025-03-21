---
title: "그래프 최단거리 구하기"
date: 2022-05-30
update: 2022-05-30
tags:
  - Algorithm
  - 다익스트라
  - 플로이드 와샬
  - 벨만포드
series: "Algorithm"
---
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/170924395-e92bdf93-a6f9-4ad4-a57a-5a996edcbe7e.png" width="70%"></p>
출처: 픽사베이

# 🚀최단 경로 탐색
지난 시간에는 그래프와 최소 신장 트리(MST)를 구하는 알고리즘인 프림, 크루스칼 알고리즘을 알아보았다. [(그래프 알고리즘이란 + 최소 신장 트리(MST))](https://ppeper.github.io/algorithm/graph/)

이번 포스팅은 그래프 최단 거리를 구하는 알고리즘에 대해서 알아보려고 한다.

## 📌다익스트라
> 다익스트라(dijkstra) 알고리즘은 그래프에서 최단 경로를 구하는 알고리즘 중 하나이다. 다익스트라 알고리즘은 도착 정점 뿐만 아니라 __하나의 정점에서 모든 다른 정점까지 최단 경로로 방문하며 각 정점까지의 최단 경로__ 를 모두 찾게 된다.

- 인접행렬을 사용: 정점의 수가 N일때, 인접행렬을 사용하면 각 단계마다 N개의 정점을 가중치가 짧은 간선을 찾기위해 순사탐색을 진행한다.
    - __시간 복잡도: O(N²)__

- 우선순위 큐 사용: 노드의 개수 V, 간선의 개수 E라고 했을때, 모든 간선을 확인하는 O(E)와 우선순위 큐에 최대로 들어갈 수 있는 수 O(E)와 추가 또는 삭제할때 드는 비용 O(LogE) -> 우선순위 큐에서의 시간 복잡도는 O(ELogE).

따라서 전체 시간 복잡도는 O(E + ELogE) = O(ELogE)이고, 보통 E <= V² 이므로, __O(ELogV)__ 라고 볼 수 있다.

### 동작 과정(우선순위 큐)
1. 모든 노드에 대한 거리를 무한대 값(충분히 큰 값)으로 초기화
2. 시작 노드에 대한 거리를 0으로 초기화 후 우선순위 큐에 담음
3. 우선순위 큐에서 가져온 노드에 대한 인접한(갈수 있는 노드)노드를 확인
4. 더 빠르면 거리를 갱신하고 우선순위 큐에 넣어줌
5. 방문 or 더 느리면 continue
6. 우선순위 큐가 empty될때까지 3~5 반복

간단하게 1,2,3번 노드를 가지는 그래프에서 거리가 갱신되는 과정을 보면 아래와 같다.

<img src="https://user-images.githubusercontent.com/63226023/169876693-1b222949-9886-4093-9aae-81bacfc62c01.png">

### 코드(우선순위 큐)
인접한 정점들에 대한 list는 graph로 생성하였다고 가정

```kotlin
data class Node(
    val end: Int,
    val weight: Int
)
// 최단 경로 리스트
val distance = IntArray(6) { Int.MAX_VALUE }

fun dijkstra(start: Int) {
    // 시작 노드의 거리는 0
    distance[start] = 0
    // 기준은 가중치로 오름차순
    val pq = PriorityQueue<Node>(compareBy { it.weight })
    // 시작 노드
    pq.add(Node(start, 0))
    while (pq.isNotEmpty()) {
        val curr = pq.poll()
        // 새로 선택된 node보다 더 가중치가 적음 -> 방문한 느드
        if (distance[curr.end] < curr.weight) {
            continue
        }
        // 현재 시작 노드의 인접 노드들 확인
        for (node in graph[curr.end]) {
            val nextNode = node.end
            // 다음 노드까지의 거리 + 현재 노드까지의 거리 -> 방문후 가는 거리
            val nextDistance = node.weight + curr.weight
            // 지금까지의 거리 보다 방문후 가는 거리가 짧으면 업데이트 후 우선순위 큐에 넣어줌
            if (nextDistance < distance[nextNode]) {
                distance[nextNode] = nextDistance
                pq.add(Node(nextNode, nextDistance))
            }
        }
    }
}
```
## 📌벨만 포드
> 다익스트라 알고리즘과 마친가지로  ___하나의 정점에서 모든 다른 정점까지 최단 경로로 방문하며 각 정점까지의 최단 거리__ 를 찾을 수 있지만 차이점은 __음의 가중치가 있는 그래프__ 에서 사용할 수 있다. 

벨만 포드 알고리즘에서는 __음수 사이클__ 로 인하여 최단 거리를 정의할 수 없는 경우도 알아낼 수 있다.

<img src="https://user-images.githubusercontent.com/63226023/170925669-279f8a63-2a15-46b5-b110-d7d2ff30a814.png">

> 1->2->3으로 가는경우 무한히 작아질 수 있으므로 최단 거리를 구할 수 없음 -> 계속 노드를 거쳐가면 1 + 2 -4 = -1 이므로 음의방향으로 발산한다.

벨만 포드 알고리즘과 다익스트라 알고리즘의 차이점은 __음수 가중치__ 가 존재하는 경우에도 적용할 수 있다는 것이며, 시간 복잡도는 __O(VE)__ (V: 정점 개수, E: 간선 대수)로 다익스트라에서 우선순위 큐를 이용한 방식인 __O(ELogV)__ 보다 느리다.

### 동작 과정
1. 모든 노드에 대한 거리를 무한대 값(충분히 큰 값)으로 초기화
2. 다음의 과정을 V-1번 반복한다.
    - 모든 간선 E개를 순서대로 확인한다.
    - 각 간선을 가쳐가 다른 노드로 가는 비용을 비교하여 최단 거리로 갱신한다.

- 음수 간선 순환이 발생하는지 확인하기 위해서는 2번의 과정을 한 번 더 수행한다. -> 최단 거리가 갱신이 된다면 음수 간선 순환이 존재한다.

### 코드
```kotlin
fun bellmanFord(start: Int): Boolean {
    // 최단 경로 리스트
    val distance = IntArray(graph.size) { INF }
    distance[start] = 0
    for (i in graph.indices) {
        for (j in graph.indices) {
            if (graph[i][j] != INF) {
                if (distance[j] > distance[i] + graph[i][j]) {
                    distance[j] = distance[i] + graph[i][j]
                    // N번째에서 갱신이 되었다면 음수 사이클 존재
                    if (i == graph.size - 1) {
                        return true
                    }
                }
            }
        }
    }
    return false
}
```

## 📌플로이드 와샬
> 플로이드 와샬 알고리즘은 벨만포드와 같이 __음의 가중치가 있는 그래프__ 에서 사용할 수 있고, 차이점은 하나의 정점이 아닌 __모든 정점에서 모든 다른 정점으로 가는 최단 거리__ 를 구할 수 있다. 다시 말하여 __다른 정점을 경유__ 하여 도착하는 최단 거리를 구할 수 있다.

플로이드-워셜 알고리즘은 DP(Dynamic Programming) 기법을 사용한 알고리즘이다. 각 정점으로 부터 __다른 정점을 순차적으로 거쳐가면서 더 짧은 길이를 선택__ 하여 갱신 한다.

다익스트라와 다른점은 정점까지의 거리 dist 배열을 __모든 정점__ 에서 모든 다른 정점으로 가는 최단 거리를 구해야 하므로 정점의 개수 V일때 V * V 크기의 dist 배열을 생성해주어야한다. 

<img src="https://user-images.githubusercontent.com/63226023/170028092-efb435f9-1df9-406a-8c81-cb1be9081667.png">

위의 거리 배열에서 각 dist[i][i]는 시작 정점으로 자기자신에 대한 거리는 0이므로 초기화 해준다.

### 동작 과정
1. 인접행렬을 저장할 2차원 배열을 만들고 무한대(충분히 큰 값)로 초기화한다.
    - 자기자신([i][i])는 0으로 초기화
    - 그래프에 따라 간선 정보를 저장한다.
2. 경유지를 거쳐가는 것이 빠르다면 거리 배열을 갱신 해준다.
    - (시작: i, 도착: j일때 k가 경유 노드라면, dist[i][j] = min(dist[i][k] + dist[k][j])로 갱신)
3. 모든 정점에 대해 순차적으로 경유지로 선택하여 2번 과정을 반복한다.

### 코드
```kotlin
fun floyd(dist: Array<IntArray>, n: Int) {
    // k -> 거쳐가는 노드
    for (k in 0 until n) {
        // i -> 출발 노드
        for (i in 0 until n) {
            // j -> 도착 노드
            for (j in 0 until n) {
                dist[i][j] = dist[i][j].coerceAtMost(dist[i][k] + dist[k][j])
            }
        }
    }
}
```
- - -

# References

- [https://born2bedeveloper.tistory.com/44](https://born2bedeveloper.tistory.com/44)
- [https://reinvestment.tistory.com/58](https://reinvestment.tistory.com/58)