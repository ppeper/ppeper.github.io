---
title: "알고리즘 DFS/BFS 탐색 방법 알아보기"
date: 2022-05-04
update: 2022-05-04
tags:
  - Algorithm
  - DFS
  - BFS
series: "Algorithm"
---
그래프에서는 `DFS`와 `BFS`방식으로 모든 정점을 탐색한다. 그래프는 따로 파트를 나누어 학습을 하고 이번 포스팅은 그래프의 탐색 방법의 기본이 되는 `DFS`와 `BFS`에 대해서 정리하려고 한다.🤔

# 깊이 우선 탐색(DFS, Depth-First Search)

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/166437047-03f6e094-c080-45d4-a98c-3754cd5f1a2d.png" width="70%"></p>

>🧷DFS는 미로에서 출구를 찾는것과 비슷하다고 볼 수 있다. 하나의 길을 갈때 쭉 진행하다가 __길이 막히면 왔던길을 되돌아 가__ 다시 안가본 길을 탐색하여 모든 경로를 탐색한다.

- 그래프에서의 DFS는 임의의 정점에서 시작하여 이웃하는 하나의 정점을 방문을 한다.
- 방금 방문한 정점의 이웃하는 정점을 방문한다.
- 이웃하는 정점들을 다 방문을 하였다면, 이전 정점으로 되돌아가 다시 탐색을 한다.

DFS는 아래와 같은 특징들이 있다.
1. DFS는 `Deep(깊게)` 탐색하는 방법이다.
2. 모든 노드를 방문을 하고자 할때 이 방법을 선택한다.
3. 검색속도는 BFS보다 보다 느리다.
4. DFS는 `Stack` 또는 `재귀함수`로 구현할 수 있다.

## DFS의 구현
DFS에서는 정점의 방문여부를 체크하기 위하여 `visited` 배열을 두어 확인한다. 재귀함수로 DFS의 구현은 아래와 같이 할 수 있다.

1. 재귀함수를 사용한 DFS 구현

```java
import java.util.Arrays;
import java.util.LinkedList;

class Graph_Recursion {
    int N;	// 그래프의 정점의 수
    LinkedList<Integer> list[];
    private boolean[ ] visited;	// DFS 수행 중 방문한 정점을 true로 만든다.

    public Graph_Recursion(int N) {
        this.N = N;
        list = new LinkedList[N];
        for (int i = 0; i < N ; i++) {
            list[i] = new LinkedList<>();
        }
        visited = new boolean[N];
        Arrays.fill(visited, false); // 배열 초기화
    }

    public void addEdge(int source, int destination){
        list[source].add(destination);
        list[destination].add(source);
    }

    private void dfs(int start) {
        for (int i = start; i < N; i++) {
            if (!visited[start]) {
                System.out.print(i + " ");
                visited[start] = true; // 점점 start를 방문함
                for (Integer node: list[start]) {
                    if (!visited[node]) {
                        dfs(node);
                    }
                }
            }
        }
    }

    public void printGraph(){
        for (int i = 0; i < N ; i++) {
            LinkedList<Integer> nodeList = list[i];
            if(!nodeList.isEmpty()) {
                System.out.print("Node = " + i + " is connected to nodes: ");
                for (Integer node : nodeList) {
                    System.out.print(" " + node);
                }
            }
            System.out.println();
        }
    }

    public static void main(String[] args) {
        Graph_Recursion graph = new Graph_Recursion(6);
        graph.addEdge(0, 1);
        graph.addEdge(0, 2);
        graph.addEdge(1, 3);
        graph.addEdge(1, 4);
        graph.addEdge(4, 5);
        graph.printGraph();
        System.out.print("Depth First Traversal: ");
        graph.dfs(0);
    }
}
```

<img src="https://user-images.githubusercontent.com/63226023/166632872-26c637e2-20d3-4246-a811-dece85be60a7.png
">

DFS의 또 다른 구현은 `Stack`을 이용하는 것이다. DFS는 `Deep(깊게)` 탐색을 먼저하였다가 다시 이전 노드의 이웃한 노드를 확인하는 방법이므로 __Stack의 Last In First Out 구조__ 를 사용하여 구현이 가능하다.

2. stack을 이용한 DFS 구현 (printGraph, addEdge 동일)

```java
class Graph_Stack {
    int N;
    LinkedList<Integer> list[];
    Stack<Integer> stack;
    private boolean[] visited; // DFS 수행 중 방문한 정점을 true로 만든다.

    public Graph_Stack(int N) {
        this.N = N;
        stack = new Stack();
        list = new LinkedList[N];
        for (int i = 0; i < N ; i++) {
            list[i] = new LinkedList<>();
        }
        visited = new boolean[N];
        Arrays.fill(visited, false); // 배열 초기화
    }

    private void dfs(int start) {
        System.out.print("Depth First Traversal: ");
        // 시작 노드
        stack.push(start);
        while (!stack.isEmpty()) {
            int node = stack.pop();
            if (!visited[node]) {
                System.out.print(node + " ");
                visited[node] = true; // 점점 node를 방문함
                LinkedList<Integer> adjList = list[node]; // 인접한 노드들 리스트
                for (int adjNode : adjList) {
                    // 인접한 노드를 탐색 안했다면 탐색
                    if (!visited[adjNode]) {
                        stack.push(adjNode);
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        Graph_Stack graph = new Graph_Stack(6);
        graph.addEdge(0, 2);
        graph.addEdge(0, 1);
        graph.addEdge(1, 4);
        graph.addEdge(4, 5);
        graph.addEdge(1, 3);
        graph.printGraph();
        graph.dfs(0);
    }
}
```

<img src="https://user-images.githubusercontent.com/63226023/166635949-38f397e5-19bf-4dda-9362-54c3e9eb431b.png">

# 너비 우선 탐색(BFS, Breadth First Search)

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/166434137-0249dea4-891f-4362-9dac-79f5a0209781.png" width="70%"></p>

>🧷BFS는 물이 퍼지는것과 같이 주변부터 진행하면서 모든 경로를 탐색한다.

- 그래프에서의 BFS는 임의의 정점에서 시작하여 이웃하는 정점을 방문을 한다.
- 방문하기 이전의 정점에서 또 다른 이웃하는 정점을 방문을 한다.
- 모든 이웃하는 정점을 방문을 하면 다시 전에 방문한 노드의 이웃한 노드를 방문 하여 탐색을 한다.

BFS는 아래와 같은 특징들이 있다.
1. BFS는 `Wide(넓게)` 탐색하는 방법이다.
2. 주로 두 노드 사이의 최단 경로를 찾고 싶을 때 사용한다.
3. DFS는 `Queue`로 구현은 한다.


## BFS의 구현

BFS는 `Wide(넓게)` 탐색을 하는 방법으로 __Queue의 First In First Out 구조__ 를 사용하여 구현이 가능하다.


1. Queue를 이용한 BFS 구현 (printGraph, addEdge 동일)

```java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.Queue;

class Graph {
    int N;  // 그래프의 정점의 수
    LinkedList<Integer> list[];
    Queue<Integer> queue;
    private boolean[] visited;    // DFS 수행 중 방문한 정점을 true로 만든다.

    public Graph(int N) {
        this.N = N;
        list = new LinkedList[N];
        queue = new LinkedList<>();
        for (int i = 0; i < N; i++) {
            list[i] = new LinkedList<>();
        }
        visited = new boolean[N];
        Arrays.fill(visited, false); // 배열 초기화
    }

    public void bfs(int start) {
        System.out.print("Breadth First Search: ");
        // 시작 노드
        queue.offer(start);
        visited[start] = true;   // 점점 node를 방문함
        // bfs 탐색
        while (!queue.isEmpty()) {
            int node = queue.poll();
                System.out.print(node + " ");
                LinkedList<Integer> adjList = list[node];   // 인접한 노드들 리스트
                for (int adjNode: adjList) {
                    // 인접한 노드들을 방문하지 않았으면 다 방문 체크 -> Wide하게 탐색 -> BFS
                    if (!visited[adjNode]) {
                        queue.offer(adjNode);
                        visited[adjNode] = true;
                    }
            }
        }
    }

    public static void main(String[] args) {
        Graph graph = new Graph(6);
        graph.addEdge(0, 1);
        graph.addEdge(0, 2);
        graph.addEdge(1, 3);
        graph.addEdge(1, 4);
        graph.addEdge(4, 5);
        graph.printGraph();
        graph.bfs(0);
    }
}
```

# References
- [https://velog.io/@lucky-korma/DFS-BFS%EC%9D%98-%EC%84%A4%EB%AA%85-%EC%B0%A8%EC%9D%B4%EC%A0%90](https://velog.io/@lucky-korma/DFS-BFS%EC%9D%98-%EC%84%A4%EB%AA%85-%EC%B0%A8%EC%9D%B4%EC%A0%90)
- [https://lemidia.github.io/algorithm/DFS-Implementation-stack/](https://lemidia.github.io/algorithm/DFS-Implementation-stack/)