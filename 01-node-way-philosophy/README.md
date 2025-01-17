# 1-1 Node.js 철학

-   경량 코어
-   경량 모듈
    -   "작은 것이 아름답다."
    -   "각 프로그램은 한 가지 역할만 잘 하도록 만들어라."

1. 이해하기 쉽고 사용하기 쉽다.
2. 테스트 및 유지보수가 쉽다.
3. 사이즈가 작아 브라우저에서 사용하기에 완벽하다.

작고 집중화된 모듈을 갖는 것은 재사용을 가능하게 한다. (DRY)

```
Keep It Simple, Stupid.
"디자인은 구현과 인터페이스 모두에서 단순해야한다. 구현이 인터페이스보다 더 단순해야 한다는 것은 더욱 중요하다. 단순함이 디자인에서 가장 중요한 고려 사항이다.

완벽하고 모든 기능을 갖춘 소프트웨어가 아닌, 단순하게 설계하는 것.
구현을 위해 적은 노력이 들고, 빨리 보급 가능하며, 유지보수가 쉽고 빠른 이해가 가능한 것. Agile!

합리적인 복잡성을 가지고 빠르게 일하는 실용적인 접근 방식!!
```

# Node.js는 어떻게 동작하는가

## I/O는 느리다.

-   컴퓨터의 기본적인 동작들 중에서 가장 느립니다.
-   CPU 측면에서는 많은 비용을 요구하지는 않지만, 요청과 작업이 완료되는 순간 사이의 지연이 발생하게 됩니다.

## 블로킹 I/O

-   전통적인 블로킹 I/O 프로그래밍에서는 I/O를 요청하는 함수의 호출은 작업이 완료될때까지 스레드의 실행을 차단합니다.
-   모든 유형의 I/O가 요청의 처리를 차단할 수 있다는 것을 생각해보면 I/O 작업의 결과를 위해서 스레드가 꽤 많이 블로킹된다는 것을 알 수 있습니다.
-   메모리를 소모하고 컨텍스트 전환을 유발하여 대부분의 시간 동안 사용하지 않는 장시간 실행 스레드를 가지게 됨 으로써 메모리와 CPU 사이클을 낭비하게 됩니다.

<b>멀티스레딩 복습하기! (컨텍스트 전환 비용, 등에 대하여)</b>

## 논 블로킹 I/O

-   시스템 호출은 데이터가 읽혀지거나 쓰여지기를 기다리지 않고 항상 즉시 반환됩니다.
-   호출 순간에 사용 가능한 결과가 없는 경우 함수는 단순히 미리 정의된 상수를 반환하여 그 순간에 사용 가능한 데이터가 없다는 것을 알립니다.
-   이러한 종류의 논 블로킹 I/O를 다루는 가장 기본적인 패턴은 실제 데이터가 반환될 때까지 루프 내에서 리소스를 적극적으로 폴링(poll)하는 것입니다.
-   폴링 방식은 사용할 수 없는 리소스를 반복하는 엄청난 CPU 시간의 낭비를 초래한다.

```
resorces = [socketA, socketB, fileA]
while(!resorces.isEmpty()) {
    for(resource of resource) {
        data = resource.read();
        if(data === 'NO_DATA') continue;
        if(data === 'CLOSED') {
            resources.remove(i)
        } else {
            consumeData(data)
        }
    }
}
```

## 이벤트 디멀티플렉싱

-   Busy-waiting은 논 블로킹 리소스 처리를 위한 이상적 기법이 아니다.
-   대부분의 운영체제는 이를 위해 동기 이벤트 디멀티플렉서 또는 이벤트 통지 인터페이스라는 메커니즘을 제공한다.
-   디멀티플렉싱은 신호가 원래의 구성요소로 다시 분할되는 작업입니다.

-   우리가 말하고 있는 동기 이벤트 디멀티플렉서는 여러 리소스를 관찰하고 이 리소스들 중에 읽기 또는 쓰기 연산의 실행이 완료되었을 때 새로운 이벤트를 반환합니다.
-   여기서 찾을 수 있는 이점은 동기 이벤트 디멀티플렉서가 처리하기 위한 새로운이벤트가 있을 때가지 블로킹 된다는 것입니다.

### 나의 생각

-   폴링은 리스트가 empty가 될 때 까지 반복문을 돈다.
-   반면 디멀티플렉서는 리소스들을 관찰하며 읽을 준비가 된 리소스가 이벤트를 보낼 때 까지 블로킹된다. (동작하지 않는다는 뜻)
-   즉.. 그냥 필요할 때만 호출한다는 것

-   여러 스레드에 작업을 분산하는 대신 시간에 따라 분산한다.
-   전체적인 유휴시간을 최소화시키는 이점이 있다.
-   싱글 스레드라는 것은 프로그래머가 동시성에 접근하는 방식에 이로운 영향을 미치게 됩니다.
-   경쟁 상태의 발생 문제와 다중 스레드의 동기화 문제가 없다는 것이 어떻게 우리에게 더 간단한 동시성 전략을 사용하게 해줄 수 있을까요?

### 리액터 패턴

-   각 I/O 작업에 연관된 핸들러(콜백함수)를 갖는다.
-   작업이 완료 되었을 때 호출될 핸들러를 제공한다.

## Libuv, Node.js의 I/O 엔진

-   Unix에서 일반 파일 시스템은 논 블로킹 작업을 지원하지 않기 때문에 논 블로킹 동작을 위해서는 이벤트 루프 외부에 별도의 스레드를 사용해야합니다.
-   서로 다른 리소스 유형의 논 블로킹 동작을 표준화 하기 위해 libuv라는 C라이브러리를 만들었다.
-   libuv는 node.js의 하위 수준의 I/O엔진을 대표하며, 가장 중요한 구성요소.
-   libuv는 기본 시스템 호출을 추상화하는 것 외에도 리액터 패턴을 구현하고 있으므로 이벤트 루프의 생성, 이벤트 큐의 관리, 비동기 I/O 작업의 실행 및 다른 유형의 작업을 큐에 담기 위한 API들을 제공합니다.

## Node.js를 위한 구성

## 운영체제 기능에 대한 모든 접근

## 네이티브 코드 실행

-   네이티브 코드와 연결될 수 있는 능력 덕분에 C/C++를 재사용 하거나, 임베디드 프로그래밍 가능
-   웹어셈블리?

## 요약

-   Node way는 더 작고 간단하며 최소한의 필요 기능만을 노출한다는 의미를 갖고 있음
-   리액터 패턴
-   다음장에서는 Node.js의 주제들 중에서 가장 근본적이고 중요한 "모듈 시스템"

GIT_COMMITTER_DATE="Jan 09 23:52:03 2023 +0900" git commit --amend --date "Jan 09 23:52:03 2023 +0900”