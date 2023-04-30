<h1> 가비지 컬렉터 </h1>

- 프로그램을 개발하다 보면 유효하지 않은 메모리인 가비지(Garbage)가 발생하게 된다.
- C언어를 사용하면 free()라는 함수를 통해 직접 메모리를 해제해주어야 한다.
- 하지만 Java Kotlin을 이용해 개발을 하다보면 개발자가 메모리를 직접 해제해주는 일이 없다.
- 그 이유는 JVM의 가비지 컬렉터가 불필요한 메모리를 알아서 정리해주기 때문이다.
- 대신 Java에서 명시적으로 불필요한 데이터를 표현하기 위해 일반적으로 NULL을 선언해준다. 
- 예를 들어 아래와 같은 코드가 있다고 가정하자.

```java

Person person = new Person();
person.setName("Mang");
person = null;

// 가비지 발생
person = new Person();
person.setName("MangKyu");

```

- 기존에 Mang으로 생성된 person 객체는 더 이상 참조를 하지 않고 사용이 되지 않아서 Garbage(가비지)가 되었다.
- Java나 Kotlin에서는 이러한 메모리 누수를 방지하기 위해 가비지 컬렉터(Garbage Colletor, GC)가 주기적으로 검사하여 메모리를 청소해준다.
- 물론 Java에서도 System.gc()를 이용해 호출할 수 있지만, 해당 메소드를 호출하는 것은 시스템의 성능에 매우 큰 영향을 미치므로 절대 호출해서는 안 된다.

<h1> Minor GC와 Major GC </h1>

- JVM의 Heap 영역은 처음 설계될 때 다음의 2가지를 전제로 설계되었다.
  - 대부분의 객체는 금방 접근 불가능한 상태(Unreachable)가 된다.
  - 오래된 객체에서 새로운 객체로의 참조는 아주 적게 존재한다.

- 즉, <b> 객체는 대부분 일회성이며, 메모리에 오랫동안 남아있는 경우는 드물다 </b>는 것이다.
- 그렇기 때문에 객체의 생존 기간에 따라 물리적인 Heap 영역을 나누게 되었고 Young, Old 총 2가지 영역으로 설계되었다.
- 초기에는 Perm 영역이 존재했지만 Java8부터 제거되었다.

![image](https://user-images.githubusercontent.com/62228401/234888374-100ff1cb-5d03-44ae-820e-69083561abd5.png)

- Young 영역(Young Generation)
  - 새롭게 생성된 객체가 할당(Allocation)되는 영역
  - 대부분의 객체가 금방 Unreacable 상태가 되기 때문에, 많은 상태가 Young 영역에 생성되었다가 사라진다.
  - Young 영역에 대한 가비지 컬렉션(Garbage Collection)을 Minor GC라고 부른다.

- Old 영역(Old generation)
  - Young 영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역
  - Young 영역보다 크게 할당되며, 영역의 크기가 큰 만큼 가비지가 적게 발생한다.
  - Old 영역에 대한 가비지 컬렉션(Garbage Collection)을 Major GC 또는 Full GC라고 부른다.

- Old 영역이 Young 영역보다 크게 할당되는 이유는 Young 영역의 수명이 짧은 객체들은 큰 공간을 필요로 하지 않으며 큰 객체들은 Young 영역이 아니라 바로 Old 영역에 할당되기 때문이다.

- 예외적인 상황으로 Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우도 존재할 것이다. 
- 이러한 경우를 대비하여 Old 영역에는 512 bytes의 덩어리(Chunk)로 되어 있는 카드 테이블(Card Table)이 존재한다.

![image](https://user-images.githubusercontent.com/62228401/234896377-e8d261bb-d7aa-4ea5-a42c-1421f23bdc12.png)

- 카드 테이블에는 <b> Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 그에 대한 정보가 표시 </b>된다.
- 카드 테이블이 도입된 이유는 간단하다.
- Young 영역에서 가비지 컬렉션(Minor GC)가 실행될 때 모든 Old 영역에 존재하는 객체를 검사하여 참조되지 않는 Young 영역의 객체를 식별하는 것이 비효율적이기 때문이다.
- 그렇기 때문에 Young 영역에서 가비지 컬렉션이 진행될 때 <b> 카드 테이블만 조회하여 GC의 대상인지 식별 </b> 할 수 있도록 하고 있다.

<h1> Garbage Collection(가비지 컬렉션)의 동작 방식 </h1>

- Young 영역과 Old 영역은 서로 다른 메모리 구조로 되어 있기 때문에, 세부적인 동작 방식은 다르다.
- 하지만 기본적으로 가비지 컬렉션이 실해된다고 하면 다음의 2가지 공통적인 단계를 따르게 된다.

1. Stop the World
2. Mark and Sweep

<h3> Stop the World </h3>

- Stop the World는 가비지 컬렉션을 실행하기 위해 <b> JVM이 애플리케이션의 실행을 멈추는 작업 </b>이다.
- GC가 실행될 때는 GC를 실행하는 쓰레드를 제외한 모든 쓰레드의 작업이 중단되고, GC가 완료되면 작업이 재개된다.
- 당연히 모든 쓰레드들의 작업이 중단되면 애플리케이션이 멈추기 때문에, GC의 성능 개선을 위해 튜닝을 한다고 하면 보통 stop-the-world 시간을 줄이는 작업을 하는 것이다.
- 또한 JVM에서도 이러한 문제를 해결하기 위해 다양한 실행 옵션을 제공하고 있다.

<h3> Mark and Sweep </h3>

- Mark : 사용되는 메모리와 사용되지 않는 메모리를 식별하는 작업
- Sweep : Mark 단계에서 사용되지 않음으로 식별된 메모리를 해제하는 작업

- Stop the World를 통해 모든 작업을 중단시키면, GC는 스택의 모든 변수 또는 Reachable 객체를 스캔하면서 각각이 어떤 객체를 참고하고 있는지를 탐색하게 된다.
- 그리고 사용되고 있는 메모리를 식별하는데, 이러한 과정을 Mark라고 한다.
- 이후에 Mark가 되지 않은 객체들을 메모리에서 제거하는데, 이러한 과정을 Sweep라고 한다.

<h1> Minor GC의 동작 방식 </h1>

- Minor GC를 정확히 이해하기 위해서는 Young 영역의 구조에 대해 이해를 해야 한다.
- Young 영역은 1개의 Eden 영역과 2개의 Survivor 영역, 총 3가지로 나누어진다.

- Eden 영역 : 새로 생성된 객체가 할당(Allocation)되는 영역
- Survivor 영역 : 최소 1번의 GC 이상 살아남은 객체가 존재하는 영역

- 객체가 새롭게 생성되면 Young 영역 중에서도 Eden 영역에 할당(Allocation)된다.
- 그리고 <b> Eden 영역이 꽉 차면 Minor GC가 발생</b> 하게 되는데, 사용되지 않은 메모리는 해제되고 Eden 영역에 존재하는 (사용중인) Survivor 영역으로 옮겨지게 된다.
- Survivor 영역은 총 2개이지만 반드시 1개의 영역에만 데이터가 존재해야 하는데, Young 영역의 동작 순서를 자세히 살펴보도록 하자.

1. 새로 생성된 객체가 Eden 영역에 할당된다.
2. 객체가 계속 생성되어 Eden 영역이 꽉 차게 되고 Minor GC가 실행된다.
  - 1. Eden 영역에서 사용되지 않은 객체의 메모리가 해제된다.
  - 2. Eden 영역에서 살아남은 객체는 1개의 Survivor 영역으로 이동된다. 
3. 1 ~ 2번의 과정이 반복되다가 Survivor 영역이 가득 차게 되면 Survivor 영역의 살아남은 객체를 다른 Survivor 영역으로 이동시킨다.(1개의 Survivor 영역은 반드시 빈 상태가 된다.)
4. 이러한 과정을 반복하여 계속해서 살아남은 객체는 Old 영역으로 이동(Promotion)된다.

- 객체의 생존 횟수를 카운트하기 위해 Minor GC에서 객체가 살아남은 횟수를 의미하는 age를 Object Header에 기록한다.
- 그리고 Minor GC 때 Object Header에 기록된 age를 보고 Promotion 여부를 결정한다.
- 또한 Survivor 영역 중 1개는 반드시 사용이 되어야 한다.
- 만약 두 Survivor 영역에 모두 데이터가 존재하거나, 모두 사용량이 0이라면 현재 시스템이 정상적인 상황이 아님을 파악할 수 있다.


<h1> Major GC의 동작 방식 </h1>

- Young 영역에서 오래 살아남은 객체는 Old 영역으로 Promotion됨을 확인할 수 있다.
- 그리고 Major GC는 <b> 객체들이 계속 Promotion되어 Old 영역의 메모리가 부족해지면 </b> 발생하게 된다.
- Young 영역은 일반적으로 Old 영역보다 크기가 작기 때문에 GC가 0,5초에서 1초 사이에 끝난다.
- 그렇기 때문에 Minor GC는 애플리케이션에 크게 영향을 주지 않는다.
- 하지만 Old 영역은 Young 영역보다 크며 Young 영역을 참조할 수도 있다.
- 그렇기 때문에 Major GC는 일반적으로 Minor GC보다 시간이 오래 걸리며, 10배 이상의 시간을 사용한다.

![image](https://user-images.githubusercontent.com/62228401/234907008-b086d545-48bc-4d02-a054-0a7c5a387977.png)
