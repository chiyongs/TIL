# Garbage Collection

## Garbage Collection이란?

메모리에서 더 이상 사용되지 않는 객체를 찾아 정리하는 것

Java 언어를 사용하면 JVM이 더 이상 사용되지 않는 객체(Garbage)를 찾아 제거해준다.

### JVM의 구조

JVM은 크게 3가지로 구성되어 있다.

- ClassLoader : 바이트코드를 읽어 메모리에 저장
- Memory
  - Memory는 크게 2가지로 분류할 수 있다.
    여러 스레드가 공유하는 영역과 아닌 영역
  - 여러 스레드가 공유하는 영역
    - Method area
    - Heap
  - 스레드 별 영역
    - Stack
    - PC Register
    - Native Method Stack
- Execution Engine
  - GC, JIT, Interpreter

GC는 JVM의 Memory 중 Heap 영역에 존재하는 미사용 객체를 제거한다.

Heap 영역에 존재하는 미사용객체는 Root space로부터 참조된다.

> Root space

Heap 영역에 대한 참조를 가지는 Root space는 Stack, Method area, Native Method Stack이다.

### GC를 구현하는 대표적인 알고리즘

- Reference counting
  - Reference counting은 Heap 영역에 할당된 객체들이 각각 reference count를 가진다
  - reference count : 각 객체를 접근할 수 있는 방법의 수
  - 기본적으로 Root space에서 접근이 가능하므로, 1로 세팅
  - reference count가 0이면 제거한다.
  - 단점
    - 순환 참조를 하는 객체이며, Root space에서의 참조가 끊긴 상태라면 reference count가 0이 아니므로 제거되지 않는다.
- Mark and Sweep
  - Reference counting의 단점을 해결하는 알고리즘
  - Root space로부터 접근 가능한 모든 객체를 객체 그래프를 통해 순회하며 표시한다. (Mark)
  - 표시가 되지 않은 객체는 더 이상 사용되지 않는 객체, unreachable 상태로 판단하여 제거한다. (Sweep)
  - 그 후 메모리를 압축한다. (Compaction)
    - Compaction은 Mark and Sweep의 필수 과정은 아님

Mark and Sweep 알고리즘의 경우 Reference counting 알고리즘과 다르게 언제 GC를 실행시킬지를 판단하여 결정해야 한다.

### GC의 성능

GC는 기본적으로 Mark, Sweep, Compaction으로 이루어지는데 이 기본 동작들이 어떻게 진행되는 지에 따라 GC의 성능이 결정된다.

## Stop the world

GC를 이야기하면 Stop the world를 빼놓을 수 없다.

Stop the world는 GC로 인해 모든 애플리케이션 스레드가 중지되는 것을 말한다.

이런 Stop the world가 길어지거나 반복되면 애플리케이션의 성능이 떨어지게 된다.

따라서, GC들은 Stop the world 시간과 횟수를 어떻게 더 줄일 수 있을지 고민하고, GC 튜닝도 Stop the world를 줄이기 위해 진행한다.

## GC를 위한 가설

- 대부분의 객체는 금방 접근 불가능 상태가 된다. (unreachable)
- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.

## GC 용어

대부분의 GC는 위 가설을 토대로 Heap을 다음과 같이 분류하여 바라본다.

- Young Generation
  - Eden
  - Survivor space
- Old Generation
- Perm

Young Generation은 Eden과 Survivor space를 같이 포함하는 영역이다.

새로운 객체 인스턴스가 생성되면 Eden에 할당된다.

만약, Young Generation 영역이 가득 찼다면 발생하는 GC를 Minor GC라 한다.

Minor GC는 Young Generation 영역을 비우며, Eden 영역에서 아직 살아있는 객체를 Survivor space 또는 Old Generation 영역으로 옮긴다.

> Survivor space 또는 Old Generation 영역으로 옮기는 기준

Survivor space 또는 Old Generation 영역으로 옮기는 기준은 age-bit에 있다.

Minor GC가 발생하면 Young Generation에서 살아남은 객체들은 가지고 있는 age-bit가 1씩 증가하게 된다.

특정 age-bit를 넘어가게 되면 Old Generation으로 옮겨진다.

이렇게 Old Generation으로 옮겨지는 것을 Promotion이라 한다.

Old Generation은 오래 살아남은 객체들이 존재하는 영역이다.

Major GC는 Old Generation 영역에서 더 이상 사용되지 않는 객체를 정리한다.

Full GC는 전체 Heap 영역을 대상으로 진행되는 GC이다.

Minor GC는 Full GC에 비해 더 자주 일어나게 되지만, 훨씬 더 작은 영역에 대해 GC를 진행하기 때문에 시간도 더 짧으며 효율적이다.

## Garbage Collector의 종류

### Serial GC

- 단일 스레드로 GC를 진행
- 메모리가 적고 단일 스레드 환경에서 효율적
- Minor, Full GC 수행 시 모든 애플리케이션 스레드가 중지

### Parallel GC

- Java 8의 Default GC, Serial GC의 Multi-Thread 버전으로 생각하면 됨
- 여러 스레드로 GC를 진행하여 Serial GC에 비해 Stop the world 시간이 짧음
- Minor, Full GC 수행 시 모든 애플리케이션 스레드가 중지

### Parallel Old GC

- Summary에 해당하는 작업이 GC를 수행한 영역에 대해서 살아있는 객체를 식별하는 작업

### CMS GC

- Concurrent Mark Sweep 의 약자
- 백그라운드 스레드로 GC를 수행하기 때문에 메모리 및 CPU 사용량이 높아짐
- Minor GC 수행 시 모든 애플리케이션 스레드가 중지되지만, Full GC 시 모든 애플리케이션 스레드를 중지하지 않고 병렬로 GC 수행
- GC 후 Compaction 과정을 진행하지 않아 메모리 단편화가 발생할 수 있음
- 메모리 단편화로 인해 Heap이 가득 차게 되면 모든 애플리케이션 스레드를 중지시키고 단일 스레드로 Full GC를 수행

### G1 GC

- Garbage First 의 약자
- Region이라는 논리적인 개념을 통해 Heap 영역을 각각의 Region 단위로 분류한다.
  Region의 상태에 따라 그 역할이 동적으로 변동된다. - Eden Region - Survivor Region - Old Region - Humonogous Region : 크기의 50%을 초과하는 큰 객체를 저장하기 위한 공간 - Available/Unused Region : 아직 사용되지 않은 Region
- 기존 GC들은 Young Generation, Old Generation 영역을 각각의 하나의 덩어리로 분류하지만, G1 GC는 영역들이 물리적으로 분리될 수 있다. (Region이라는 논리적 개념으로 생각하기 때문)
- Minor GC 수행 시 모든 애플리케이션 스레드 중지
- 주기적으로 병렬로 Heap 영역의 미사용 객체를 관리
- 적은 Heap을 가지고 있다면 제 성능을 발휘하지 못하고 Full GC가 발생한다.
- G1 GC의 주요 동작
  - Minor GC
  - Concurrent Cycle
  - Mixed GC
  - Full GC

> Minor GC

G1 GC의 Minor GC는 Eden 영역이 가득차면 촉발된다.

Minor GC 후 Eden 영역은 전부 비워지며 Survivor space은 적어도 1개가 존재하게 되며, 일부 데이터는 Old Generation으로 이동된다.

이 과정에서 Stop the world가 발생한다.

> Concurrent Cycle

G1 GC는 백그라운드 스레드를 사용해 Concurrent Cycle을 진행한다.

Concurrent Cycle은 다음과 같은 단계로 진행된다.

- Initial Mark Phase, 초기 표시 단계
  - Old Region에 존재하는 객체들이 참조하는 Survivor Region을 찾고 표시한다.
  - Survivor Region을 탐색해야 하므로 모든 애플리케이션 스레드를 중지시킬 필요가 있다.
  - 따라서, Minor GC 주기를 이용하여 Initial Mark Phase를 시작함.
- Root Region Scan
  - Initial Mark Phase에서 표시된 Survivor 객체들에 대한 스캔 작업을 실시한다.
  - 이 단계에서는 애플리케이션 스레드를 중지하지 않으며, 백그라운드 스레드만 사용한다.
  - 따라서, 가용 CPU 주기 확보가 중요하다.
  - 왜냐하면, 해당 단계는 Minor GC에 의해 중단될 수 없으므로 Root Region Scan 도중 Young Generation이 가득 차 Minor GC가 필요하게 되면 Root Region Scan이 종료된 후에 Minor GC을 실행하게 된다. → Minor GC로 인한 애플리케이션 스레드 중단 시간이 더 길어지게 된다.
- Concurrent Marking Phase
  - 전체 Heap의 scan 작업을 실시하고 GC 대상 객체가 발견되지 않은 Region은 이후 단계를 제외한다.
  - 해당 단계는 중단될 수 있으므로 Minor GC가 이 단계에서 일어난다.
- Remarking Phase
  - 애플리케이션 스레드를 중지시키고, 최종적으로 GC 대상에서 제외할 객체를 식별한다.
- Cleanup Phase
  - 애플리케이션 스레드를 중지시키고, 살아있는 객체가 가장 적은 Region부터 미사용 객체를 제거한다.
- Copy Phase
  - GC 대상의 Region이었지만, Cleanup 과정에서 완전히 비워지지 않은 Region의 살아남은 객체들을 새로운 Region에 복사하여 Compaction을 수행한다.

> Mixed GC

Minor GC를 수행하면서, 백그라운드 조사로 표시된 영역들에서 Garbage Collection 작업이 이루어지기 때문에 혼합 GC라 일컫음 (Young & Old Region을 모두 GC하기 때문에)

> G1GC의 Full GC 촉발 주요 시기

1. Concurrent Mode Failure
   1. Concurrent Marking 단계나 다른 병행 GC 단계 도중 Old Generation이 해당 단계가 완료되기 전에 꽉 찬 경우
   2. Heap 크기를 늘리거나 G1GC의 백그라운드 처리를 좀 더 빨리 시작하거나, cycle이 더 빨리 수행되도록 튜닝해야 한다.
2. Promotion 실패
   1. Mixed GC를 시작했지만, Old Region으로 승격이 필요한 상황에 Old Generation에 충분한 공간이 존재하지 않는 경우
   2. Mixed GC가 좀 더 빨리 발생할 필요가 있다는 사실을 의미하며, Minor GC가 Old Generation 내에서 더 많은 영역을 처리할 필요가 있다.
3. 비우기 실패
   1. Minor GC 수행 시 Survivor space와 Old Generation 내 살아남은 객체를 전부 유지하기 위한 공간이 충분하지 않은 경우
   2. 주로 Heap이 가득 찼거나 단편화됐다는 지표.
   3. 가장 쉽게 해결하는 방법은 Heap size를 늘리는 것
4. 대규모 할당 실패 (Humonogous 객체 처리 실패)
   1. Humonogous 객체를 할당시킬 여유 공간이 없는 경우
   2. 표준 GC 로그에서 명확하게 해당 상황을 진단할만한 도구가 없다.
