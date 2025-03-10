---
title: "Thread와 Coroutine 짚고 넘어가기"
date: 2022-02-07
update: 2022-02-07
tags:
  - OS
  - 비동기
series: "OS"
---
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/152731810-a0d2ddc8-cbee-48b0-8398-3b6deb4f0233.png"></p>

> - 안드로이드 앱은 `UI 스레드가 너무 오랫동안 차단`되면 'ANR(애플리케이션 응답 없음)' 오류가 발생하게 된다❗
> - 앱에서 개발을 하다보면 __비동기 백그라운드 작업__ 을 통하여 이러한 ANR이 발생하지 않도록 노력을 해야한다. 

처음으로 앱을 개발할때 네트워크 통신을 할때 위와 같은 문제에 봉착하여 큰 난관에 빠졌었던 기억이 많다. 이번 포스팅을 시작으로 __비동기 작업__ 을 공부하며 정리해 보려고 한다🤔

📍 잠깐 짚고 넘어가기
> __동기(synchronous : 동시에 일어나는)__ : 요청을 보내면 결과값이 오기까지 작업을 멈춘다.   
> __비동기(Asynchronous : 동시에 일어나지 않는)__ : 요청을 보내면 작업을 멈추지 않고 다른 일을 수행한다.
- - -

# Thread 개요
스레드는 멀티태스킹 운영체제의 초석으로 __메인 프로세스 내__ 에서 __작은 프로세스가 실행__ 되는 것으로 생각할 수 있다. 

> 📍스레드의 목적은 앱 내부에서 __병렬로 실행될 수 있는 코드__ 를 만드는 것

# 🤔알아보고 가기
## Process vs Thread
프로세스와 비슷해보이는 스레드를 알아보기전에 `~ 프로그램`이라는 말을 많이 들어봤을 것이다.   
프로그램의 예시는 흔히 위도우 컴퓨터에서 볼 수 있는 __.exe__ 파일이라고 생각하면 된다.
> 📍Program : __보조기억장치에 존재하는 실행파일__

이러한 `프로그램`은 사용자가 클릭을 하여 운영체제가 `실행`을 하게 해야한다. 즉 실행되고 있는 프로그램이 __프로세스__ 라고 할 수 있다.

 <p align="center"><img src="https://user-images.githubusercontent.com/63226023/152743857-ac10d808-8324-461b-9f97-40a07e1eeadb.png"></p>

> 📍Process : __메모리에 올라와 실행되고 있는 프로그램__ 의 인스턴스

🔔프로세스는 다음과 같은 특징이 있다.
 - 프로세스는 하나 이상의 스레드(메인 스레드)를 포함한다.
 - 각 프로세스는 별도의 주소 공간에서 실행되며, 한 프로세스는 다른 프로세스의 변수나 자료구조에 접근할 수 없다.
 - 다른 프로세스의 자원에 접근을 하려면 프로세스간 통신(IPC: Inter-Process-Communication)을 사용해야 한다.
 - 프로세스는 각각 __Code, Data, Stack, Heap의 구조__ 로 되어있는 __독립된 메모리 영역__ 을 할당 받는다.

 - - -

 그렇다면 하나의 예시로 크롬창(하나의 프로세스)으로 웹 서핑을 하고 있다고 생각해 보자. 사용자들은 백그라운드로 게임을 다운하면서 웹 서핑을 계속 하고싶을 것이다. 이러한 __하나의 프로세스에서 더 작인 실행 단위__ 를 여러 __스레드__ 를 통하여 실행하게 된다.

 <p align="center"><img src="https://user-images.githubusercontent.com/63226023/152743769-aa8ae95c-6af9-4369-98f1-92504df1bb76.png"></p>

 > 📍Thread : 독립적인 메모리공간(Stack)을 가지는 __프로세스 내에서 실행되는 여러 흐름 단위__

 🔔스레드는 다음과 같은 특징이 있다.
 - 하나의 스레드는 __독립적인 메모리공간(Stack)__ 을 할당 받고 프로세스의 Code, Data, Heap 영역은 공유한다.
 - 각 스레드는 Stack 메모리를 공유할 수 없다.

# 동시성 vs 병렬성
Thread와 코루틴은 모두 __동시성 프로그래밍__ 을 위한 기술이다.
## 동시성 (Concurrency)
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/152747561-c8870560-b07d-403a-8cbb-695a0fba3cce.png"></p>

동시성 프로그래밍은 여러 작업을 __동시에 실행__ 하는 것이다. 말은 동시에 실행이라고 하였지만 이는 __작업들을 잘게 쪼게어 빠르게 번갈아가며 수행__ 하여 두 작업은 __동시에 실행되는 것__ 처럼 느껴지게 하는 것이다.

## 병렬성 (Parallelism)
<p align="center"><img src="https://user-images.githubusercontent.com/63226023/152748798-19726566-bf04-4b27-9453-cd23b31f6b21.png"></p>

병렬성 프로그래밍은 말 그대로 여러 작업을 __한 번에 동시에 실행__ 하는 것이다. 즉 병렬성은 실행시키는 자원(CPU 코어)이 여러개(멀티코어)일때 가능한 방법이다.

# Thread vs Coroutine
Thread와 Coroutine은 둘다 __동시성을 보장__ 하기 위해 사용된다. 처음 코루틴에 대한 용어를 알게되어 찾아봤을때는 스레드와 별반 차이를 몰랐었다. 스레드와 코루틴의 개념을 알아보면서 둘의 차이점을 알아보았다.

## Thread
- 작업의 단위 -> __Thread__
    - Thread는 독립적인 메모리공간(Stack)을 가진다
- 동시성 보장 수단: __Context Switching__
    - OS 커널에 의한 __Context Switching__ 을 사용한다.
    - 블로킹 : (Thread 1) 이 (Thread 2) 의 결과가 나오기까지 기다려야한다면 (Thread 1)은 __블로킹__ 되어 그 시간동안 해당 자원을 사용하지 못한다.

> __🤔Context Switching?__   
> - CPU에서 여러 프로세스를 번갈아 가면서 작업을 처리하는 데 이 과정을 Context Switching라 한다.   
> - 구체적으로, 동작중인 프로세스가 대기 할때 해당 프로세스의 정보(Context)를 보관하고 있다가 다음 순서의 프로세스가 동작할때 해당 정보(Context)를 통하여 상태를 복구하는 작업을 말한다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/152752310-17cd2826-0ad2-476f-aa85-e059b251d86b.png"></p>

위의 그림을 보면 작업 단위는 모두 Thread인것을 볼 수 있다. 위의 그림에서 __동시성이 보장__ 이 되는 과정을 보자.

- Thread A 에서 작업 1을 수행중에 작업 2가 필요할때 이를 __비동기로 호출__ 하게 된다.
- 이 때 Thread A 는 __작업을 멈추고(블로킹)__ 되고, __Thread B 로 Context Switching__ 이 일어나 작업 2 를 수행한다.
- 작업 2가 완료되면 해당 결과값을 __Thread A 로 Context Switching__ 이 일어나 작업 1에 반환하게 되고, 동시에 수행할 작업 3과 작업 4는 각각 Thread C 와 Thread D 에 할당된다.
- OS 커널에서는 __Preempting Scheduling__ 을 통하여 각 작업에 대해 __얼만큼 수행할지 또는 어떤 작업을 먼저 수행할지를 결정__ 하여 그에 맞게 동작을 하게 하여 __동시성을 보장__ 하게 되는 것이다.

## Coroutine
- 작업의 단위 -> __Object(Coroutine)__
    - 여러 작업에 각각 __Object__ 를 할당한다.
    - __Coroutine Object__ 는 객체를 담는 __JVM Heap__ 에 적재됩니다.
- 동시성 보장 수단: __Programmer Switching__ = ~~No Context Switching~~
    - __프로그래머의 코딩__ 을 통해 Switching 시점을 마음대로 정한다.
    - __Suspend__ (Non-Blocking): `Object 1(작업 1)`이 `Object 2(작업 2)`의 __결과가 나오기까지 기다려야한다__ 면
    `Object 1(작업 1)`는 __Suspend(중지)__ 되지만 `Object 1(작업 1)`를 수행하던 __Thread 는 그대로 유효__ 하기 때문에 `Object 2(작업 2)` 도 `Object 1(작업 1)` 과 __동일한 Thread 에서 실행__ 될 수 있다.

<p align="center"><img src="https://user-images.githubusercontent.com/63226023/152752334-57ccdaa1-cea6-43d4-a17f-72444f42a64d.png"></p>

작업의 단위는 __Coroutine Object__ 이므로 작업 1 수행중에 비동기 작업 2가 발생하더라도 작업 1을 수행하던 같은 __Thread에서 작업 2를 수행할 수 있으며__, __하나의 Thread__ 에서 __여러개의 Coroutine Object__ 들을 수행할 수도 있다.

위의 그림에서 작업 1에서 작업 2의 전환에 있어서 __Thread A 위에서 Coroutine Object 객체들만 교체__ 만  이뤄지기 때문에 Context Switching 은 필요가 없다.   
Coroutine은 한 Thread 에 __다수의 작업(Objcet)__ 을 수행할 수 있음과 __Context Switching 이 필요없기__ 때문에 __Lightweight Thread__ 라고도 불린다.

다만 위의 세번째 그림에서 처럼 작업 3, 작업 4를 위해 Thread C가 동시에 실행이 된다면, __동시성을 보장__ 하기 위하여 Context Switching이 필요하다. 따라서 Coroutine의 __No Context Switching 이라는 장점__ 을 최대한 활용하기 위해 ~~여러 Thread 를 사용하는 것~~보다 __단일 Thread 에서 여러 Coroutine Object 들을 실행하는 것__ 이 좋다😮

> 💡Coroutine 은 ~~Thread 의 대안~~이 아니라 __기존의 Thread__ 를 __더 잘게 쪼개어 사용__ 하기위한 개념이다.   
>    - 작업의 단위가 Thread가 아닌 Object 단위로 축소하면서 __단일 Thread가 여러 코루틴을 다룰 수 있기 때문__ 에 작업의 수만큼 Thread를 생성하지 않아도 되어 __메모리 낭비, Context Switching__ 의 비용 낭비를 할 필요가 없다.


# 마무리 정리
처음 Coroutine의 용어를 접했을 때는 Thread와의 차이를 알지 못하고 같은 작업을 한다고 생각하였었다😂

앱 개발에서 __비동기 작업__(DB 트랜잭션, 네트워킹 요청등)은 거의 필수적으로 사용한다고 생각한다. 이러한 __비동기 작업__ 은 앞서 정리한 내용과 같이 __Coroutine을 활용__ 하여 비용절감을 할 수 있기 때문에 안드로이드 공부를 꾸준히 해가면서 많이 접하고 적극적으로 사용해봐야겠다👍

- - -

# References

- [https://velog.io/@haero_kim/Thread-vs-Coroutine-%EB%B9%84%EA%B5%90%ED%95%B4%EB%B3%B4%EA%B8%B0#process-with-thread]()
- [https://gmlwjd9405.github.io/2018/09/14/process-vs-thread.html]()
- [https://velog.io/@gparkkii/ProgramProcessThread]()