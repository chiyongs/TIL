# HashTable

## 개념

Key 와 Value를 1:1로 연관지어 저장하는 자료구조
-> Key를 이용하여 Value를 도출

## 기능

- Key, Value 쌍을 저장
- Key가 주어진 경우, 해당 Key에 대한 Value 조회
- 기존 Key에 새로운 Value가 주어진 경우, 기존 Value를 새로운 Value로 대체
- Key로 해당 Key에 대한 Value 삭제

## 구조

<img src="/해시/images/해시테이블.png">

- Key
  - 고유한 값
  - 저장 공간의 효율성을 위해 Hash Function을 통해 Hash로 변경 후 저장
- Hash Function
  - Key를 Hash로 바꿔주는 역할
  - 해시 충돌이 발생할 확률을 최대한 줄이는 함수를 만드는 것이 중요
    > 해시 충돌 : 서로 다른 Key가 동일한 Hash를 갖게 되는 경우
- Hash
  - Hash Function의 결과
  - 저장소에서 Value와 매칭되어 저장
- Value
  - 저장소에 최종적으로 저장되는 값
  - Key와 매칭되어 저장, 삭제, 검색 가능

### 동작 과정

Key를 Hash Function을 통해 Hash로 만들고, 이 Hash를 배열의 index로 사용하는 Bucket에 저장

## 해시 충돌

서로 다른 Key가 동일한 Hash를 갖게 되는 경우
-> 충돌이 많아질수록 탐색의 시간 복잡도가 O(1)에서 O(n)으로 증가하게 됨

### 해시 충돌 해결 방법

- Seperating Chaining
  - JDK 내부에서 사용하는 해시 충돌 처리 방식
    <img src="/해시/images/Seperating Chaining.png">
  - 충돌 발생 시 충돌이 발생한 인덱스가 가리키는 LinkedList에 노드를 추가하여 Value 삽입
  - 충돌 데이터가 6개 이하인 경우 : LinkedList
  - 충돌 데이터가 8개 이상인 경우 : Red-Black Tree
    > 자료구조의 변경이 6개 이하, 8개 이상으로 7개인 경우를 따로 지정하지 않는 이유는 버퍼를 두어 과도한 자료구조의 변경이 발생하지 않도록 오버헤드를 막는 것
- Open Addressing
  - 추가 메모리 공간을 사용하지 않고, HashTable 배열의 빈 공간을 사용하는 방법
  - Seperating Chaining에 비해 적은 메모리를 사용
  - Linear Probing, Quadratic Probing...
- Resizing
  - 저장 공간이 일정 수준 채워지면 크기를 재조정하는 작업
  - 보통 2배로 확장
  - Seperating Chaining의 경우 성능 향상을 위해 수행
  - Open Addressing의 경우 배열 크기 확장을 위해 수행
  - Resizing이 실행되는 인계점 : 현재 데이터 개수가 Hash Bucket 개수의 75%가 될 때

## 장단점

장점

- 적은 리소스로 많은 데이터를 효율적으로 관리 가능
- 배열의 인덱스를 사용하기 때문에 빠른 검색, 삽입, 삭제 속도 제공
- Key와 Hash에 연관성이 존재하지 않아 보안에 유리

단점

- 해시 충돌이 발생할 수 있음 -> 관리 포인트 증가
- 공간 복잡도 증가
- 해시 함수에 의존
