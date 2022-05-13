---
title: 그리디 알고리즘(Greedy Algorithm)에 대해
categories: Algorithm
tags: ['Algorithm', '탐욕법']
header:
    teaser: /assets/teasers/greedy-algorithm.png
last_modified_at: 2022-4-24T00:00:00+09:00
---
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/164942840-d52e6623-5973-4c16-99ef-3da83ce1e9ab.png"></p>

- - -
# 그리디(탐욕)
알고리즘에서 그리디(탐욕법) 알고리즘이란 이름에서 유추해 볼 수 있듯이 __현재 상황에서 가장 최선의 선택__ 을 하는 알고리즘을 말한다.

매 순간마다 하는 선택은 그 순간에 대해 `지역적`으로는 최적이지만, 그 선택들을 계속 수집하여 최종적(`전역적`)인 해답을 만들었다고 해서, 그것이 최적의 해답이라는 보장은 없다. 

하지만 그리디 알고리즘을 적용할수 있는 문제들은 `지역적`으로 최적이면 `전역적`으로 최적인 문제들이다. 
- - -

# 어떤 경우에 잘 작동하는가?
> 🎯그리디 알고리즘이 적용되기 위해서는 아래와 같은 조건이 만족되야 한다.

- 탐욕스런 선택 조건
- 최적 부분 구조 조건

> 📍탐욕스런 선택 조건(greedy choice property)은 __앞의 선택이 이후의 선택에 영향을 주지 않는다__ 는 것이다.
>
> 📍최적 부분 구조 조건(optional substructure)은 __문제에 대한 최적해__ 가 __부분 문제에 대해서도 최적해라는 것__ 이다.

- - -
# 거스름돈 문제🪙
그리디 문제중에서 거스름돈 문제가 있다. 어떤 금액에 대해 거스름돈을 받을때, 동전을 최소의 개수로 받는 문제이다.

단, 거스름돈 문제가 항상 __그리디 알고리즘__ 으로 해결되는 것은 아니다.   
우리의 실생활에서의 동전은 10, 50, 100, 500원 이 있지만, 문제에서 다음과 같이 냈다고 하자.

> 거스름돈 14원을 주려고한다. 동전은 1, 6, 10원이 있다고하자.

위의 문제를 그리디 알고리즘을 적용하면 __가장 큰 단위의 동전을 선택하여 거스롬든을 주자__ 로 생각해 볼 수 있고 구하는 해는 10원, 1원 * 4 -> 5가지가 된다. 반면에 최소의 개수는 6원 * 2, 1원 * 2 -> 4가지이다.

이 거스름돈 문제가 그리디 알고리즘이 적용되기 위해서는 __각각의 거스름돈이 서로의 배수/약수의 관계__ 가 되어야 한다. 따라서 실생활에 사용되는 동전으로 이루어진 거스름돈 문제는 그리디 알고리즘이 적용이 가능하다.

> (500원 = 100원 5개, 100원 = 50원 2개, 50원 = 10원 5개) -> 작은 단위 동전을 조합하여 다른 해가 나올 수 없기 때문에 그리디 알고리즘 사용이 가능하다.

이렇게 그리디 알고리즘을 적용하기 위해서는 문제의 요구사항을 파악하고 위에서 적용되는 조건을 확인하는것이 중요하다.🤔
- - -
# References

- [https://ko.wikipedia.org/wiki/탐욕_알고리즘](https://ko.wikipedia.org/wiki/%ED%83%90%EC%9A%95_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98)
- [https://ujink.tistory.com/10](https://ujink.tistory.com/10)