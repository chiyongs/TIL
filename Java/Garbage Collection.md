Garbage Collection에 대해 알기 전 `stop-the-world` 용어를 알아야 한다.

> stop-the-world

Garbage Collection(이하 GC)을 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것

`stop-the-world` 가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈추고 GC 작업을 완료한 후에 중단했던 작업을 다시 시작한다.

모든 GC 알고리즘은 `stop-the-world` 가 발생하고 대개의 GC 튜닝은 이 `stop-the-world` 시간을 줄이는 것

### GC를 위한 가설

- 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
