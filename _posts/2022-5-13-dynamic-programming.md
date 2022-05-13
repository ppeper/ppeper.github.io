---
title: 알고리즘 Dynamic Programming(동적 프로그래밍) 살펴보기
categories: Algorithm
tags: ['Dynamic Programming']
header:
    teaser: /assets/teasers/dynamic-programming-image.png
last_modified_at: 2022-5-13T00:00:00+09:00
---
# Dynamic Programming(동적 프로그래밍)
동적 프로그래밍이란 __주어진 문제를 부분 문제로 나누어__ 각 부분 문제의 답을 계산하고, __이 계산한 결과값을 이용해 원래 문제의 답을 산출__ 하는 방법이다.

## Divide And Conquer

동적 프로그래밍과 같이 큰 문제를 작은 문제로 나누는 알고리즘은 __분할 정복(Divide And Conquer)__ 이 있다.
> 📌분할 정복(Divide And Conquer)
>
> __Divide(분할):__ 주어진 문제를 부분 문제로 나눈다.
>
> __Conquer(정복):__ 작은 문제들을 더 이상 분할되지 않을때까지 __분할(recursion)__ 후 작은문제에 대한 답을 구한다.
>
> __Conbine(결합):__ 나누어진 작은 문제에 대한 정복된 답을 결합을 통하여 __원래의 문제에 대한 답을 구한다__

> 🧷분할 정복은 주어진 문제를 작은 문제로 나누어 푸는 방식으로 __하향식(Top down)__ 접근 방법이다.

- - -
## Dynamic Programming

동적 프로그래은 분할 정복과 다르게 __작은 문제가 반복__ 되는지에 대한 여부가 중요하다. 동적 프로그래밍은 작은 문제의 반복되는 값을 __메모를 해 놓았다가__ 다시 사용하여 푸는 방법이다.

1. 동적 프로그래밍을 사용하기 위한 조건은 아래와 같다.
> - 문제를 나누었을 때 작은 문제가 __반복적으로__ 일어나는 경우
>
> - 같은 나누어진 문제에 대한 __답이 항상 같을 경우__

2. Memoization

앞서 동적 프로그래밍은 __작은 문제가 반복되는 부분을 메모를 해 놓았다가 사용__ 한다고 하였다. 이를 Memoization이라고 표현 하며 배열을 통하여 저장할 공간을 생성한다.
> 🧷Memoization: 계산한 결과를 저장하고 필요할때 다시 사용한다.

# 피보나치
동적 프로그래밍의 예시를 위하여 피보나치 수열을 보면 다음과 같다.

> 피보나치 수: 첫 번째 및 두번 째 항이 1이며 그 뒤의 모든 항은 바로 앞 두 항의 합인 수열

1. Recursion 사용

```kotlin
fun fibonacci(n: Int): Int {
    if (n <=1) {
        return n
    }
    return fibonacci(n - 1) + fibonacci(n - 2)
    // fibonacci(7) -> 13
}
```
피보나치 수열을 구하는 과정을 보면 아래와 같다.

<img src="https://user-images.githubusercontent.com/63226023/168243379-d011f728-5ed4-4696-bfc0-193a44890949.png">

피보나치에서 F4까지만 보게되면 여기서도 이미 __F3은 3번, F4는 3번은 반복되어 사용__ 되는 것을 볼 수 있다.
> Fibonacci(n) = Fibonacci(n-2) + Fibonacci(n-1)

이를 동적 프로그래밍으로 `Memoization`을 사용하여 작은 문제에 대한 결과값을 저장하여 사용하면 아래와 같다.

전역 변수 `memo`의 배열에서 값을 저장해두고 필요할 때 사용하여 값을 구한다.

```kotlin
val memo = IntArray(100) { 0 }
fun fibonacci(n: Int): Int {
    return if (n <= 1) {
        memo[n] = n
        memo[n]
    } else {
        if (memo[n] == 0) {
            memo[n] = fibonacci(n - 1) + fibonacci(n - 2)
        }
        memo[n]
    }
    // fibonacci(7) -> 21
}
```
- - -