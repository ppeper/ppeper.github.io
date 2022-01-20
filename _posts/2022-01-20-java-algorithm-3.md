---
title: 프로그래머스 2022 KAKAO RECRUITMENT - 신고 결과 받기
categories: Algorithm
tags: ['Android', 'programmers', '2022 KAKAO RECRUITMENT']
header:
    teaser: /assets/teasers/programmers.png
last_modified_at: 2022-1-20T00:00:00+09:00
---
- - -
### 문제 설명
신입사원 무지는 게시판 불량 이용자를 신고하고 처리 결과를 메일로 발송하는 시스템을 개발하려 합니다. 무지가 개발하려는 시스템은 다음과 같습니다.

* 각 유저는 한 번에 한 명의 유저를 신고할 수 있습니다.
    * 신고 횟수에 제한은 없습니다. 서로 다른 유저를 계속해서 신고할 수 있습니다.
    * 한 유저를 여러 번 신고할 수도 있지만, 동일한 유저에 대한 신고 횟수는 1회로 처리됩니다.

* k번 이상 신고된 유저는 게시판 이용이 정지되며, 해당 유저를 신고한 모든 유저에게 정지 사실을 메일로 발송합니다.
    * 유저가 신고한 모든 내용을 취합하여 마지막에 한꺼번에 게시판 이용 정지를 시키면서 정지 메일을 발송합니다.

다음은 전체 유저 목록이 ["muzi", "frodo", "apeach", "neo"]이고, k = 2(즉, 2번 이상 신고당하면 이용 정지)인 경우의 예시입니다.

<img src = "https://user-images.githubusercontent.com/63226023/150295190-4e88d4cf-f7ce-44c5-992e-1341efe30350.png">

각 유저별로 신고당한 횟수는 다음과 같습니다.

<img src = "https://user-images.githubusercontent.com/63226023/150295269-95f0c657-f21e-43c3-a6c0-b2e7f0f3dc96.png">

위 예시에서는 2번 이상 신고당한 "frodo"와 "neo"의 게시판 이용이 정지됩니다. 이때, 각 유저별로 신고한 아이디와 정지된 아이디를 정리하면 다음과 같습니다.

<img src = "https://user-images.githubusercontent.com/63226023/150295291-4a52f6d3-482f-4215-8393-8a08d0566bee.png">

따라서 "muzi"는 처리 결과 메일을 2회, "frodo"와 "apeach"는 각각 처리 결과 메일을 1회 받게 됩니다.

이용자의 ID가 담긴 문자열 배열 `id_list`, 각 이용자가 신고한 이용자의 ID 정보가 담긴 문자열 배열 `report`, 정지 기준이 되는 신고 횟수 `k`가 매개변수로 주어질 때, 각 유저별로 처리 결과 메일을 받은 횟수를 배열에 담아 return 하도록 solution 함수를 완성해주세요.

### 제한사항
* 2 ≤ `id_list`의 길이 ≤ 1,000
    * 1 ≤ `id_list`의 원소 길이 ≤ 10
    * `id_list`의 원소는 이용자의 id를 나타내는 문자열이며 알파벳 소문자로만 이루어져 있습니다.
    * `id_list`에는 같은 아이디가 중복해서 들어있지 않습니다.

* 1 ≤ `report`의 길이 ≤ 200,000
    * 3 ≤ `report`의 원소 길이 ≤ 21
    * `report`의 원소는 "이용자id 신고한id"형태의 문자열입니다.
    * 예를 들어 "muzi frodo"의 경우 "muzi"가 "frodo"를 신고했다는 의미입니다.
    * id는 알파벳 소문자로만 이루어져 있습니다.
    * 이용자id와 신고한id는 공백(스페이스)하나로 구분되어 있습니다.
    * 자기 자신을 신고하는 경우는 없습니다.

* 1 ≤ `k` ≤ 200, `k`는 자연수입니다.
    * return 하는 배열은 `id_list`에 담긴 id 순서대로 각 유저가 받은 결과 메일 수를 담으면 됩니다.

### 입출력 예

<img src = "https://user-images.githubusercontent.com/63226023/150296370-6f07b381-c923-436c-b870-43a2a89539fd.png">

### 풀이
가장 처음 신고내용이 중복이 안된다고 하였으므로 집합(Set)을 사용하여 중복된 신고를 빼야한다고 생각하였다. 
```java
HashSet<String> reportFilter = new HashSet<String>(Arrays.asList(report));
```
그 이후 reportFilter를 가지고 각 인원의 신고당한 횟수를 저장하고 이미 신고를 당했으면 HashMap에 __getOrDefault()__ 를 통하여 +1을 해주었다.

각 인원이 신고한 사람들을 따로 리스트를 만들어서 저장을 하려하다가 __getOrDefault()__ 함수로 신고를 한사람이 2명 이상이면 __" "__ 에 다음 신고자를 저장하는 방법을 사용해 보았다.

```java
    // 신고내용 저장 && 신고당한사람 Count
    private void setReports(HashMap<String, String> hm, HashMap<String, Integer> reportCount, HashSet<String> reportFilter) {
        for (String str : reportFilter) {
            String[] split = str.split(" ");
            // 신고자가 신고한 사람들 저장 -> 이미 신고한사람이 있으면 그 뒤에 " "후 다음 신고한 인원 저장
            hm.put(split[0], hm.getOrDefault(split[0], "") + " " +split[1]);
            // 신고당한 사람 Count -> 이미 신고를 당했으면 +1
            reportCount.put(split[1], reportCount.getOrDefault(split[1], 0) + 1);
        }
    }
```

신고자를 매개변수로 받아서 그 사람이 신고한 사람들이 __k__ 이상으로 신고당했으면 count를 하여 return 하는 함수를 사용하였다.

> -> 여기서 처음 신고자의 문자열을 저장할때 __" "__ 을 저장으로 시작하였으므로 split 한후에 1번 인덱스부터 확인하였다. (0번은 ""(빈문자열))

```java
    private int count(String person, HashMap<String, String> hm, HashMap<String, Integer> reportCount, int k) {
        int count = 0;
        // 신고자가 신고한 리스트 가져옴
        String str = hm.getOrDefault(person, "");
        String[] list = str.split(" ");
        for (int i = 1; i < list.length; i++) {
            String check = list[i];
            if (reportCount.getOrDefault(check, 0) >= k) count++;
        }
        return count;
    }
```

__전체 코드__

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;

class Report {
    public int[] solution(String[] id_list, String[] report, int k) {
        int[] answer = new int[id_list.length];
        HashSet<String> reportFilter = new HashSet<String>(Arrays.asList(report));
        HashMap<String, String> hm = new HashMap<>();
        HashMap<String, Integer> reportCount = new HashMap<>();
        setReports(hm, reportCount, reportFilter);

        // 결과값 구하기
        for (int i = 0; i < id_list.length; i++) {
            answer[i] = count(id_list[i], hm, reportCount, k);
        }

        return answer;
    }

    // 신고내용 저장 && 신고당한사람 Count
    private void setReports(HashMap<String, String> hm, HashMap<String, Integer> reportCount, HashSet<String> reportFilter) {
        for (String str : reportFilter) {
            String[] split = str.split(" ");
            // 신고자가 신고한 사람들 저장 -> 이미 신고한사람이 있으면 그 뒤에 " "후 다음 신고한 인원 저장
            hm.put(split[0], hm.getOrDefault(split[0], "") + " " +split[1]);
            // 신고당한 사람 Count -> 이미 신고를 당했으면 +1
            reportCount.put(split[1], reportCount.getOrDefault(split[1], 0) + 1);
        }
    }

    private int count(String person, HashMap<String, String> hm, HashMap<String, Integer> reportCount, int k) {
        int count = 0;
        // 신고자가 신고한 리스트 가져옴
        String str = hm.getOrDefault(person, "");
        String[] list = str.split(" ");
        for (int i = 1; i < list.length; i++) {
            String check = list[i];
            if (reportCount.getOrDefault(check, 0) >= k) count++;
        }
        return count;
    }
}
```

