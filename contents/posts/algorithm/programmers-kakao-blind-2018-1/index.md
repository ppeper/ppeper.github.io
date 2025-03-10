---
title: "프로그래머스 2018 KAKAO RECRUITMENT - [1차]캐시"
date: 2022-02-05
update: 2022-02-05
tags:
  - Algorithm
  - programmers
series: "Algorithm"
---
### 문제 설명

지도개발팀에서 근무하는 제이지는 지도에서 도시 이름을 검색하면 해당 도시와 관련된 맛집 게시물들을 데이터베이스에서 읽어 보여주는 서비스를 개발하고 있다.
이 프로그램의 테스팅 업무를 담당하고 있는 어피치는 서비스를 오픈하기 전 각 로직에 대한 성능 측정을 수행하였는데, 제이지가 작성한 부분 중 데이터베이스에서 게시물을 가져오는 부분의 실행시간이 너무 오래 걸린다는 것을 알게 되었다.
어피치는 제이지에게 해당 로직을 개선하라고 닦달하기 시작하였고, 제이지는 DB 캐시를 적용하여 성능 개선을 시도하고 있지만 캐시 크기를 얼마로 해야 효율적인지 몰라 난감한 상황이다.

어피치에게 시달리는 제이지를 도와, DB 캐시를 적용할 때 캐시 크기에 따른 실행시간 측정 프로그램을 작성하시오.

### 입력 형식
* 캐시 크기(`cacheSize`)와 도시이름 배열(`cities`)을 입력받는다.
* `cacheSize`는 정수이며, 범위는 0 ≦ `cacheSize` ≦ 30 이다.
* `cities`는 도시 이름으로 이뤄진 문자열 배열로, 최대 도시 수는 100,000개이다.
* 각 도시 이름은 공백, 숫자, 특수문자 등이 없는 영문자로 구성되며, 대소문자 구분을 하지 않는다. 도시 이름은 최대 20자로 이루어져 있다.

### 출력 형식
* 입력된 도시이름 배열을 순서대로 처리할 때, "총 실행시간"을 출력한다.

### 제한사항
* 캐시 교체 알고리즘은 `LRU`(Least Recently Used)를 사용한다.
* `cache hit`일 경우 실행시간은 `1`이다.
* `cache miss`일 경우 실행시간은 `5`이다.

### 입출력 예
<img src="https://user-images.githubusercontent.com/63226023/152626198-05c7d28f-2480-4a3b-945d-c974c35638d4.png">

### 풀이
문제에서 요구하는 __LRU__ 캐시 교환 알고리즘은 __가장 최근에 사용된 적이 없는 캐시의 메모리부터 대체하며 새로운 데이터로 갱신__ 하는 방법이다. 가장 사용안됨 Last -> 새로 교체는 FIFO구조로 보아 __Queue__ 를 사용하였다. 

- 캐시가 hit면 1을 더하고 데이터를 삭제후 다시 캐시에 넣어줘서 업데이트
- 캐시가 miss면 5를 더함
    - 캐시 크기가 남아있다면 그냥 데이터를 캐시에 넣어줌
    - 캐시가 꽉 차있다면 `캐시 교환`을 해줌

# 코드

```java
import java.util.LinkedList;
import java.util.Queue;

class Solution {
    public int solution(int cacheSize, String[] cities) {
        int answer = 0;
        if (cacheSize == 0) {
            return answer += 5 * cities.length;
        }
        Queue<String> LRU = new LinkedList<>();
        for (String str : cities) {
            String city = str.toUpperCase();
            if (LRU.isEmpty()) {
                LRU.offer(city);
                answer += 5;
            } else {
                // cache hit!
                if (checkCache(LRU, city)) {
                    answer += 1;
                    LRU.remove(city);
                    LRU.offer(city);
                } else { // cache miss!
                    if (LRU.size() == cacheSize) {
                        LRU.poll();
                    }
                    LRU.offer(city);
                    answer += 5;
                }
            }
        }
        return answer;
    }
    // 캐쉬에 해당 도시가 있으면 true
    private boolean checkCache(Queue<String> LRU, String city) {
        return LRU.contains(city);
    }
}
```
