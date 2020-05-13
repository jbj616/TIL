## Java Garbage Collection

> Naver D2 기사 요약 https://d2.naver.com/helloworld/1329

#### 가비지 컬렉션 과정

- stop - the - world : GC을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것
  - stop the world 실행되면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드가 멈춘다. (이 시간을 줄이는 게 GC의 핵심)
- Java에서 메모리를 명시적으로 해제하지 않음 -> 가비지 컬렉션이 쓰레기 객체를 찾아 지우는 작업을 해야함
  - 가비지 컬렉션의 두가지 조건(가설) 하에 만들어졌음. (weak generational hypothesis)
    - 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
    - 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
- HotSpot VM에서 두개의 물리적 공간으로 나누었다.
  - Young 영역 : 새롭게 생성된 객체들이 위치하는 곳, 대부분의 객체가 금방 접근 불가능 상태가 된다. 객체가 사라질 때 Minor GC가 발생한다고 한다.
  - Old 영역 : 접근 불가능 상태로 되지 않아 Young 영역에서 살아남은 객체가 복사된다. Young영역 보다 크게 할당되며 GC가 적게 발생한다. 객체가 사라질 때 Major GC라고 한다

![JavaGarbage1](https://d2.naver.com/content/images/2015/06/helloworld-1329-1.png)

- Card Table
  - Old 영역에서 Young 영역을 참조할 때 정보를 저장

![JavaGarbage2](https://d2.naver.com/content/images/2015/06/helloworld-1329-2.png)



#### Young 영역의 구성

- Eden 영역
- Survivor 영역(2개)

처리 순서

- 새로 생성한 대부분의 객체는 Eden 영역에 위치
- Eden 영역에서 GC가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동
- Eden 영역에서 GC가 발생하면 이미 살아남은 객체가 존재하는 Survivor 영역으로 객체가 계속 쌓임
- 하나의 Survivor 영역이 가득 차게 되면 그 중에서 살아남은 객체를 다른 Survivor 영역으로 이동한다. 그리고 가득 찬 Survivor 영역은 아무 데이터도 없는 상태로 된다.
- 과정을 반복하다가 계속해서 살아남아 있는 객체는 Old 영역으로 이동

![JavaGarbage3](https://d2.naver.com/content/images/2015/06/helloworld-1329-3.png)



빠른 메모리 할당을 위해서 사용하는 기술

- bump-the-pointer
  - Eden 영역에 할당된 마지막 객체를 추적
  - 생성되는 객체가 있으면, 해당 객체의 크기가 Eden 영역에 넣기 적당한지만 확인한다. 만약 해당 객체의 크기가 적당하다고 판정되면 Eden 영역에 넣게 되고, 새로 생성된 객체가 맨 위에 있게 된다. 따라서, 새로운 객체를 생성할 때 마지막에 추가된 객체만 점검하면 되므로 매우 빠르게 메모리 할당이 이루어진다.
- TLABs(Thread-Local Allocation Buffers)
  - Thread-Safe하기 위해서 만약 여러 스레드에서 사용하는 객체를 Eden 영역에 저장하려면 락(lock)이 발생할 수 밖에 없고, lock-contention 때문에 성능은 매우 떨어짐
  - 각각의 스레드가 각각의 몫에 해당하는 Eden 영역의 작은 덩어리를 가질 수 있도록 하는 것이다. 각 쓰레드에는 자기가 갖고 있는 TLAB에만 접근할 수 있기 때문에, bump-the-pointer라는 기술을 사용하더라도 아무런 락이 없이 메모리 할당




