# Java 8

## 자바 8을 눈여겨봐야 하는 이유

자바 8은 프로그램을 더 효과적이고 간결하게 구현할 수 있는 새로운 개념과 기능을 제공한다.

기존의 자바 프로그래밍 기법으로는 멀티코어 프로세서를 온전히 활용하기 어렵다.

함수는 일급값이다. 메서드를 어떻게 함수형값으로 넘겨주는지, 익명 함수를 어떻게 구현하는지 기억하자.

자바 8의 스트림 개념 중 일부는 컬렉션에서 가져온 것이다.

스트림과 컬렉션을 적절하게 활용하면 스트림의 인수를 병렬로 처리할 수 있으며 더 가독성이 좋은 코드를 구현할 수 있다.

인터페이스에서 디폴트 메서드를 이용하면 메서드 바디를 제공할 수 있으므로 인터페이스를 구현하는 다른 클래스에서 해당 메서드를 구현하지 않아도 된다.

함수형 프로그래밍에서 null 처리 방법과 패턴 매칭 활용 등 흥미로운 기법을 발견할 수 있었다.

## 동작 파라미터화 코드 전달하기

동적 파라미터를 추가 → 쓸데없는 코드가 늘어남 → 자바 8은 람다 표현식으로 해결

동적 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.

이를 통해 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있다.

자바 8 이전에는 코드를 지저분하게 구현해야 했다.

익명 클래스로 어느정도 깔끔하게 코드를 구현할 수 있지만 자바 8에서는 인터페이스 상속으로 여러 클래스를 구현해야 하는 수고를 없앴고 람다 표현식으로 더 간결하게 구현할 수 있게 되었다.

## 람다 표현식

람다 표현식 : 메서드로 전달할 수 있는 익명 함수를 단순화한 것

### 람다의 특징

- 익명 : 보통의 메서드와 달리 이름이 없어 익명이라 표현, 구현해야 할 코드에 대한 걱정이 줄어듬
- 함수 : 메서드처럼 특정 클래스에 종속되지 않음
- 전달 : 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있음
- 간결셩 : 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요 X

### 람다 기본 문법

람다의 구성요소는 파라미터 리스트, 화살표, 람다의 바디 이렇게 세 가지 부분으로 이루어진다.

- (parameter) → expression
- (parameter) → {statements;}

### 함수형 인터페이스

함수형 인터페이스 : 정확히 하나의 추상메서드를 지정하는 인터페이스

많은 디폴트 메서드가 있더라도 추상메서드가 오직 1개면 함수형 인터페이스

> 왜 함수형 인터페이스에만 람다 표현식이 사용 가능할까?

언어 설계자들이 자바에 함수형식을 추가하는 방법도 고려했지만 언어를 더 복잡하게 만들지 않기 위해 현재 방법을 선택했다.

`@FunctionalInterface` : 함수형 인터페이스임을 가리키는 어노테이션, 실제로 함수형 인터페이스가 아니면 컴파일러가 에러를 발생시킨다.

### 람다 활용 : 실행 어라운드 패턴

실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태

```java
public static String processFile() throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return br.readLine();
	}
}
```

```java
public interface BufferedReaderProcess {
	String process(BufferedReader br) throws IOException;
}

public static String processFile(BufferedReaderProcess p) {
	try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
		return p.process(br);
	}
}
```

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 함수형 인터페이스 사용

함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다.

함수형 인터페이스의 추상 메서드 시그니처를 함수 디스크립터라고 한다.

- Predicate<T> : 제네릭 형식 T의 객체를 인수로 받아 불린을 반환
  - `boolean test(T t)` : T 형식의 객체를 사용하는 불린 표현식
- Consumer<T> : 제네릭 형식 T 객체를 받아서 void를 반환
  - `void accept(T t)` : T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때
  - forEach에 Consumer를 인자로 사용할 수 있음
- Function<T, R>
  - `R apply(T t)` : 제네릭 형식 T를 인수로 받아서 제니릭 형식 R 객체를 반환하는 추상 메서드
  - 예) 사과의 무게 정보를 추출 or 문자열을 길이와 매핑

예제

- Predicate<List<String>> : (List<String> list) → list.isEmpty(), boolean
- Supplier<Apple> : () → new Apple(10), () → T
- Consumer<Apple> : (Apple a) → System.out.println(a.getWeight()), (T) → void
- Function<String, Integer> or ToIntFunction<String> : (String s) → s.length(), T → R
- Comparator<Apple> … : (int a, int b) → a \* b

## 람다의 지역 변수 사용

람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수를 활용할 수 있다.

이것을 람다 캡처링이라고 한다.

하지만, 지역 변수는 명시적으로 final로 선언되어 있어야 하거나 실질적으로 final로 선언된 변수와 똑같이 사용되어야 한다.

> 이유 : 지역변수는 스택에 위치하기 때문에 값을 직접 제공하지 않고 복사본을 제공한다.
> 따라서, 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다.

## 메서드 레퍼런스

```java
// 기존 코드
inventory.sort((Apple a1, Apple a2) ->
								a1.getWeight().compareTo(a2.getWeight()));

// 메서드 레퍼런스 활용 코드 Apple::getWeight
inventory.sort(comparing(Apple::getWeight));
```

메서드 레퍼런스는 특정 메서드만을 호출하는 람다의 축약형이라고 생각할 수 있다.

- 명시적으로 메서드명을 참조함으로써 가독성 향샹
- 메서드명 앞에 구분자(::)를 붙이는 방식으로 활용
- Apple::getWeight == (Apple a) → a.getWeight()

예시

- () → Thread.currentThread().dumpStack() == Thread.currentThread::dumpStack
- (str, i) → str.substring(i) == String::substring
- (String s) → System.out.println(s) == System.out::println

### 메서드 레퍼런스 만드는 방법

메서드 레퍼런스는 3 가지 유형으로 구분된다.

1. 정적 메서드 레퍼런스
   1. Integer.parseInt → Integer::parseInt
   2. (args) → ClassName.staticMethod(args) == ClassName.staticMethod
2. 다양한 형식의 인스턴스 메서드 레퍼런스
   1. String.length → String::length
   2. (arg0, rest) → arg0.instanceMethod(rest) == ClassName::instanceMethod
3. 기존 객체의 인스턴스 메서드 레퍼런스
   1. 변수명::메서드명
   2. (args) → expr.instanceMethod(args) == expr::instanceMethod

### 생성자 레퍼런스

생성자 레퍼런스 문법 = ClassName::new

## 람다, 메서드 레퍼런스 활용하기

```java
// 1
public class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2) {
		retrun a1.getWeight().compareTo(a2.getWeight());
	}
}
inventory.sort(new AppleComparator());

// 2
inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) {
		return a1.getWeight().compareTo(a2.getWeight());
	}
});

// 3
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 4
inventory.sort(comparing((a) -> a.getWeight()));

// 5
inventory.sort(comparing(Apple::getWeight));
```

이처럼 메서드 레퍼런스를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.

또햔, Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 이쓴ㄴ 다양한 디폴트 메서드를 제공한다.

- Comparator
  - thenComparing
  - reversed
- Predicate
  - and
  - or
- Function
  - andThen
  - compose

## 스트림

컬렉션 : 데이터를 그룹화하고 처리할 수 있다.

- 대부분 프로그래밍 작업에 필수적인 요소

스트림 : 자바 API에 새로 추가된 기능

- 선언형(데이터를 처리하는 임시 구현 코드 대신 질의로 표현)으로 컬렉션 데이터를 처리
  - 더 간결하고 가독성이 좋아진다.
- 멀티 스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리
  - 성능이 좋아진다.
- 조립할 수 있음
  - 유연성이 좋아진다.

### 스트림의 정의

> 스트림

데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- 연속된 요소
  - 스트림은 컬렉션과 마찬가지로 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공
  - 컬렉션은 자료구조이므로 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룸
  - 스트림은 filter, sorted, map처럼 표현 계산식이 주를 이룸
  - 따라서, 컬렉션의 주제 : 데이터, 스트림의 주제 : 계산
- 소스
  - 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비
  - 리스트 → 스트림으로 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지
- 데이터 처리 연산
  - 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산 지원
  - filter, map, reduce, find, match, sort
  - 스트림 연산은 순차적으로 또는 병렬로 실행할 수 있음
- 파이프라이닝
  - 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환
  - 이를 통해 laziness, short-circuiting 같은 최적화 가능
- 내부 반복
  - 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부 반복 지원
  - 데이터 소스가 연속된 요소를 스트림에 제공
  - 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환 (collect 제외)
    - filter
      - 람다를 인수로 받아 스트림에서 특정 요소를 제외시킴
    - map
      - 람다를 이용해서 한 요소를 다른 요소로 변환하거나 정보를 추출
    - limit
      - 정해진 개수 이상의 요소가 스트림에 저장되지 못하게 스트림 크기를 축소
    - collect
      - 스트림을 다른 형식으로 변환

## 스트림과 컬렉션

컬렉션과 스트림의 가장 큰 차이 : 데이터를 언제 계산하느냐

- 컬렉션 : 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조 → 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 함
  - 컬렉션은 적극적으로 생성(생산자 중심 : 팔기도 전에 창고를 가득 채움)
- 스트림 : 이론적으로 요청할 때만 요소를 계산하는 고정된 자료구조
  - 사용자가 요청하는 값만 스트림에서 추출한다는 것이 핵심 → 사용자는 이러한 변화를 알 수 없다.
  - 스트림은 생산자와 소비자 관계를 형성하며 게으르게 만들어지는 컬렉션과 같다. → 사용자가 데이터를 요청할 때만 계산

스트림은 단 한 번만 소비할 수 있다.

> 스트림과 컬렉션의 철학적 접근

스트림 : 시간적으로 흩어진 값의 집합

컬렉션 : 특정 시간에 모든 것이 존재하는 공간에 흩어진 값

### 외부 반복과 내부 반복

외부 반복(external iteration) : 컬렉션 인터페이스를 사용하려면 사용자가 직접 요소를 반복해야 함

내부 반복(internal iteration) : 스트림 라이브러리는 반복을 알아서 처리하고 결과 스트림값을 어딘가에 저장해주는 내부 반복 사용

내부 반복의 장점 : 작업을 투명하게 병렬로 처리, 더 최적화된 다양한 순서로 처리 가능

## 스트림 연산

중간 연산(intermediate operation) : 연결할 수 있는 스트림 연산

최종 연산(terminal operation) : 스트림을 닫는 연산

### 중간 연산

중간 연산의 특징 : 단말 연산을 스트림 파이프라인에 실행하기 전까지는 아무 연산도 수행하지 않는다는 것 → lazy

중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리

### 최종 연산

스트림 파이프라인에서 결과를 도출

### 스트림 이용 과정

- 질의를 수행할 데이터 소스
- 스트림 파이프라인을 구성할 중간 연산 연결
- 스트림 파이프라인을 실행하고 결과를 만들 최종 연산

스트림 파이프라인의 개념은 빌더 패턴과 비슷

중간 연산과 최종 연산 예시

- 중간 연산
  - filter, map, limit, sorted, distinct
- 최종 연산
  - forEach, count, collect

## 스트림 요약

- 스트림 : 소스에서 추출된 연속 요소 → 데이터 처리 연산을 지원
- 내부 반복을 지원
- 중간 연산과 최종 연산
- 스트림의 요소는 요청할 때만 계산됨

## 필터링과 슬라이싱

- 프레디케이트로 필터링
  - filter 메서드 : 프레디케이트를 인수로 받아서 프레디케이드와 일치하는 모든 요소를 포함하는 스트림을 반환
- 고유 요소 필터링
  - distinct 메서드 : 고유 요소로 이루어진 스트림을 반환
- 스트림 축소
  - limit(n) 메서드 : 주어진 사이즈 이하의 크기를 갖는 새로운 스트림을 반환
- 요소 건너뛰기
  - skip(n) 메서드 : 처음 n개 요소를 제외한 스트림을 반환 (n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출 시 빈 스트림이 반환)

## 매핑

스트림 API인 map과 flapMap 메서드는 특정 객체에서 특정 데이터를 선택하는 작업을 수행한다.

- 스트림은 함수를 인수로 받는 map 메서드를 지원
- 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑
- 기존의 값을 고친다는 개념보다 새로운 버전을 만든다라는 개념에 가까워 변환이 아닌 매핑이라는 단어를 사용

### 스트림 평면화

- flatMap을 사용하면 하나의 평면화된 스트림을 얻을 수 있다.
- flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다.

## 검색과 매칭

- 특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리
  - anyMatch : 프레디케이트가 적어도 한 요소와 일치하는지 확인
    - 불린을 반환 → 최종 연산
  - allMatch : 프레디케이트가 모든 요소와 일치하는지 검사
  - noneMatch : 프레디케이트와 일치하는 요소가 없는지 확인
  - findAny : 현재 스트림에서 임의의 요소르 반환
    - 다른 스트림 연산과 연결해서 사용 가능
  - findFirst : 연속된 데이터로부터 생성된 스트림에서 첫 번째 요소를 찾음
    - findAny와 findFirst 둘다 존재하는 이유 : 병렬 실행에서는 첫 번째 요소를 찾기 어렵기 때문

> 쇼트서킷 평가

전체 스트림을 처리하지 않았더라도 결과를 반환할 수 있다.

예시로 여러 and 연산으로 연결된 커다란 불린 표현식의 경우 하나라도 거짓이라면 나머지 표현식의 결과와 상관없이 거짓이 된다.

이러한 상황을 쇼트서킷이라고 부른다.

> Optional

Optional<T> : 값의 존재나 부재 여부를 표현하는 컨테이너 클래스

- isPresent() : Optional이 값을 포함하면 true, 포함하지 않으면 false 반환
- ifPresent(Consumer<T> block) : 값이 있으면 주어진 블록을 실행
- T get() : 값이 존재하면 값을 반환하고, 없으면 NoSuchElementException 일으킴
- T orElse(T other) : 값이 있으면 값을 반환, 없으면 기본값 반환

## 리듀싱

리듀싱 연산 : 모든 스트림 요소를 처리해서 값으로 도출하는 연산

- 함수형 프로그래밍 언어 용어 : 폴드(종이를 작은 조각이 될 때까지 반복해서 접는 것과 비슷하다는 의미)

### 요소의 합

```java
int sum = 0;
for (int x : numbers) {
	sum += x;
}

// 람다
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
// 정적 메서드
int sum = numbers.stream().reduce(0, Integer::sum);
```

reduce는 두 개의 인수를 가짐

- 초깃값
- 두 요소를 조합해서 새로운 값을 만드는 식

초기값을 받지 않도록 오버로드된 reduce도 존재 → Optional 객체 반환

- 스트림에 아무 요소도 없는 상황이면 초깃값이 없으므로 합계를 반환할 수 없음 → Optional 객체로 감싼 결과를 반환

### 최댓값과 최솟값

- 최댓값
  - Optional<Integer> max = numbers.stream().reduce(Integer::max);
- 최솟값
  - Optional<Integer> min = numbers.stream().reduce(Integer::min);

### 스트림 연산

- 상태 없음 (내부 상태를 갖지 않는 연산, stateless operation)
  - 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다.
  - map, filter…
- 상태 있음 (내부 상태를 갖는 연산, stateful operation)
  - 내부 상태를 가지고 있어야 연산이 가능함
  - reduce, sum, max…

## 숫자형 스트림

스트림 요소의 합 같은 연산을 진행하다보면 `Integer::sum` 을 사용해야 하는데 이 때는 박싱 비용이 존재한다.

이를 효율적으로 처리하기 위해 기본형 특화 스트림(`primitive stream specialization`)을 제공한다.

- 기본형 특화 스트림
  - 특화 스트림은 오직 박싱 과정에서 일어하는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지 않는다.
  - IntStream, mapToInt
    - OptionalInt 존재 : 스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 구분하기 위함
  - DoubleStream, mapToDouble
    - OptionalDouble
  - LongStream, mapToLong
    - OptionalLong
  - 객체 스트림으로 복원

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
Stream<Integer> stream = intStream.boxed();
```

## 컬렉터

멀티레벨로 그룹화를 수행할 때 명령형 프로그래밍과 함수형 프로그래밍의 차이점이 더욱 두드러진다.

### 고급 리듀싱 기능을 수행하는 컬렉터

훌륭하게 설계된 함수형 API의 장점 : 높은 수준의 조합성과 재사용성

collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점 → 컬렉터의 최대 강점

보통 함수를 요소로 변환할 때 컬렉터를 적용하며 최종 결과를 저장하는 자료구조에 값을 누적한다.

Collectors에서 제공하는 메서드의 기능

- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

- counting()

```java
long howManyDishes = menu.stream().collect(Collectors.counting());

long howManyDishes = menu.stream().count();
```

### 리듀싱 연산

- Collectors.maxBy
  - 스트림의 최댓값 계산
- Collectors.minBy
  - 스트림의 최솟값 계산

```java
Comparator<Dish> disCaloriesComparator =
		Comparator.comparingInt(Dish::getCalories);

Optional<Dish< mostCalorieDish =
		menu.stream()
				.collect(maxBy(dishCaloriesComparator));
```

### 요약 연산

- Collectors.summingInt, summingLong, summingDouble : 합계
  ```java
  int totalCalories = menu.stream()
  											.collect(Collectors.summingInt(Dish::getCalories));
  ```
- Collectors.averagingInt, averagingLong, averagingDouble : 다양한 형식으로 이루어진 숫자 집합의 평균 계산
  ```java
  double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
  ```
- Collectors.summarizingInt, summarizingLong, summarizingDouble
  - 컬렉터로 스트림의 요소 수를 계산, 최댓값과 최솟값을 찾고, 합계와 평균 모두 산출
  - IntSummaryStatistics: {count=9, sum=4300, min=120, average=477.7778, max=800}
  ```java
  IntSummaryStatistics menuStatistics =
  		menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));
  ```

### 문자열 연결

- Collectors.joining
  - 스트림의 각 객체에 toString 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환

```java
String shortMenu = menu.stream()
		.map(Dish::getName).collect(Collectors.joining());
// 결과 : porkbeefchickenfrench friesriceseason fruitpizzaprawnssalmon

// 결과 문자열을 해석할 수 없음
// 연결된 두 요소 사이에 구분 문자열을 넣을 수 있음

String shortMenu = menu.stream()
		.map(Dish::getName).collect(Collectors.joining(", "));
// 결과 : pork, beef, chicken, french fries, rice ....
```

### 범용 리듀싱 요약 연산

- Collectors.reducing
  - 위에 사용한 특화된 컬렉터를 모두 reducing으로 구현 가능
  - but, 가독성과 편의성이 중요하므로 특화된 컬렉터를 사용했음
  ```java
  int totalCalories = menu.stream().collect(
  													Collectors.reducing(0, Dish::getCalories, (i, j) -> i + j));
  ```
  - reducing : 3 개의 인수
    1. 첫 번째 인수 : 리듀싱 연산의 시작값 or 스트림에 인수가 없을 때의 반환값
    2. 두 번째 인수 : 변환 함수
    3. 세 번째 인수 : 실행할 연산
  - reducing : 1개의 인수를 가질 때도 있음
    - 이 때는 스트림의 첫 번째 요소를 시작 요소, 자신을 그대로 반호나하는 항등함수를 두 번째 인수로 받음

> collect와 reduce 차이

```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
List<Integer> numbers = stream.reduce(
																new ArrayList<Integer>(),
																(List<Integer> l, Integer e) -> {
																					l.add(e);
																					return l; },
																(List<Integer> l1, List<Integer> l2) -> {
																					l1.addAll(l2);
																					return l1; });
```

collect : 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드

reduce : 두 값을 하나로 도출하는 불변형 연산

→ 위 예제에서 reduce 메서드는 누적자로 사용된 리스트를 변환시키므로 reduce를 잘못 활용한 예

> 자신의 상황에 맞는 최적의 해법 선택

함수형 프로그래밍에서는 하나의 연산을 다양한 방법으로 해결할 수 있다.

또한, 스트림 인터페이스에서 직접 제공하는 메서드를 이용하는 것에 비해 컬렉터를 이용하는 코드가 더 복잡하다.

코드가 더 복잡한 대신 재사용성과 커스터마이징을 제공하는 높은 수준의 추상화와 일반화를 얻을 수 있다.

문제를 해결할 수 있는 다양한 해결 방법을 확인한 다음에 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다.

이렇게 한다면 가독성과 성능이라는 두 마리 토끼를 잡을 수 있다.

## 그룹화

데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 데이터베이스에서 많이 수행되는 작업

- 분류 함수

  - Collectors.groupingBy

  ```java
  // 요구사항이 단순한 경우
  Map<Dish.Type, List<Dish>> dishesByType =
  										menu.stream().collect(Collectors.groupingBy(Dish::getType));

  // 더 복잡한 분류 기준이 필요한 상황 -> 람다를 사용하여 구현
  public enum CaloricLevel { DIET, NORMAL, FAT}

  Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
  		Collectors.groupingBy(dish -> {
  				if (dish.getCalories() <= 400) {
  						return CaloricLevel.DIET;
  				} else if (dish.getCalories() <= 700) {
  						return CaloricLevel.NORMAL;
  				} else {
  						return CaloricLevel.FAT;
  				}
  }));
  ```

  - groupingBy를 사용하여 다수준의 그룹화부터 서브그룹으로 데이터 수집도 가능하다.
  - 분류 함수 한 개의 인수를 갖는 groupingBy(f) == groupingBy(f, toList())의 축약형
  - Collectors.collectingAndThen
    - 적용할 컬렉텨와 변환 함수를 인수로 받아 다른 컬렉터를 반환
    - 반환되는 컬렉터는 기존 컬렉터의 래퍼 역할을 하며 collect의 마지막 과정에서 변환함수로 자신이 반환하는 값을 매핑한다.
  - Collectors.mapping
    - groupingBy와 자주 사용
    - 스트림의 인수를 변환하는 함수와 변환 함수의 결과 객체를 누적하는 컬렉터를 인수로 받음
    - 입력 요소를 누적하기 전에 매핑 함수를 적용해서 다양한 형식의 객체를 주어진 형식의 컬렉터에 맞게 변환하는 역할

## 분할

- 분할 : 분할 함수라 불리는 프레디케이트를 분류 함수로 사용하는 특수한 그룹화 기능
- 분할 함수는 불린을 반환하므로 맵의 키 형식은 Boolean이다.

```java
Map<Boolean, List<Dish>> partitionedMenu =
						menu.stream().collect(Collectors.partitioningBy(Dish::isVegetarian));
```

- 분할의 장점
  - 분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것

## Collector 인터페이스

Collector 인터페이스는 리듀싱 연산(즉, 컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

ex) toList, groupingBy

Collector 인터페이스를 구현하는 리듀싱 연산을 만들 수 있다.

```java
public interface Collector<T, A, R> {
		Supplier<A> supplier();
		BiConsumer<A, T> accumulator();
		Function<A, R> finisher();
		BinaryOperator<A> combiner();
		Set<Characteristics> characteristics();
}
```

- T : 수집될 스트림 항목의 제네릭 형식
- A : 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
- R : 수집 연산 결과 객체의 형식 (대개 컬렉션 형식)

- Supplier : 새로운 결과 컨테이너 만들기

  - 빈 결과로 이루어진 Supplier 반환
  - 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수

    ```java
    public Supplier<List<T>> supplier() {
    		return () -> new ArrayList<T>();
    }

    public Supplier<List<T>> supplier() {
    		return ArrayList::new;
    }
    ```

- Accumulator : 결과 컨테이너에 요소 추가하기

  - 리듀싱 연산을 수행하는 함수를 반환
  - 스트림에서 n번째 요소를 탐색할 때 누적자와 n번째 요소를 함수에 적용
  - 함수의 반환값 : void → 요소를 탐색하면서 적용하는 함수에 의해 누적자 내부 상태가 바뀌므로 누적자가 어떤 값일지 단정 X

    ```java
    public BiConsumer<List<T>, T> accumulator() {
    		return (list, item) -> list.add(item);
    }

    public BiConsumer<List<T>, T> accumulator() {
    		return List::add;
    }
    ```

- Finisher : 최종 변환값을 결과 컨테이너로 적용하기
  - 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수를 반환
    ```java
    public Function<List<T>, <List<T>> finisher() {
    		return Function.identity();
    }
    ```
- Combiner : 두 결과 컨테이너 병합
  - 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할지 정의
    ```java
    public BinaryOperator<List<T>> combiner() {
    		return (list1, list2) -> {
    				list1.addAll(list2);
    				return list1;
    		}
    }
    ```
- Characteristics : 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환
  - 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공
    - Characteristics의 3 항목
      - UNORDERED
        - 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
      - CONCURRENT
        - 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있음
        - 컬렉터의 플래그에 UNORDERED와 함께 설정되지 않았다면 데이터 소스가 정렬되어 있지 않은 상황에서만 병렬 리듀싱 수행 가능
      - IDENTITY_FINISH
        - finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐 → 생략 가능
        - 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용 가능
        - 누적자 A를 결과 R로 안전하게 형변환 가능

# 리팩토링, 테스팅, 디버깅

## 가독성과 유연성을 개선하는 리팩토링

람다 표현식은 익명 클래스보다 코드를 좀 더 간결하게 만들고 더 큰 유연성을 갖출 수 있다.

람다 표현식을 이용한 코드는 다양한 요구사항 변화에 대응할 수 있도록 동작을 파라미터화한다.

### 코드 가독성 개선

일반적으로 코드 가독성이 좋다 → 어떤 코드를 다른 사람도 쉽게 이해할 수 있음을 의미

자바 8의 코드 가독성에 도움을 주는 기능

- 코드의 장황함을 줄여서 쉽게 이해할 수 있는 코드를 구현할 수 있음
- 메서드 레퍼런스와 스트림 API를 이용해서 코드의 의도를 쉽게 표현할 수 있음

### 익명 클래스를 람다 표현식으로 리팩토링

익명 클래스로 만들어지는 장황한 코드를 람다 표현식으로 간결하고 가독성이 좋은 코드로 변경할 수 있다.

하지만, 모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.

1. 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 가짐
   1. 익명 클래스에서 this : 익명 클래스 자신을 가리킴, 람다에서 this : 람다를 감싸는 클래스를 가리킴
2. 익명 클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다, 람다는 불가능

   ```java
   int a = 10;
   // 컴파일 에러
   Runnable r1 = () -> {
   		int a = 2;
   		System.out.println(a);
   };
   // 정상 작동
   Runnable r2 = new Runnable() {
   		public void run() {
   				int a = 2;
   				System.out.println(a);
   		}
   };
   ```

3. 익명 클래스를 람다 표현식으로 바꾸면 컨텍스트 오버로딩에 따른 모호함이 초래될 수 있다.
   1. 동일한 시그니처를 갖는 함수형 인터페이스의 경우 모호함이 발생할 수 있다.
   2. 하지만, 현재 대부분의 IDE에서 제공하는 리팩토링 기능을 이용하면 문제 해결 가능

### 람다 표현식을 메서드 레퍼런스로 리팩토링

람다 표현식도 쉽게 전달할 수 있는 짧은 코드이지만 메서드 레퍼런스를 이용하면 가독성을 더 높일 수 있다.

메서드 레퍼런스의 메서드명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.

comparing과 maxBy 같은 정적 헬퍼 메서드 또는 Collectors API를 사용하면 코드의 의도가 좀 더 명확해진다.

### 명령형 데이터 처리를 스트림으로 리팩토링

스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다.

- short circuit, lazyness → 강력한 최적화 & 멀티코어 아키텍처를 활용할 수 있는 지름길 제공

```java
List<String> dishNames = new ArrayList<>();
for(Dish dish : menu) {
		if (dish.ggetCalories() > 300) {
				dishNames.add(dish.getName());
		}
}
```

→

```java
menu.parallelStream()
		.filter(d -> d.getCalories() > 300)
		.map(Dish::getName)
		.collect(toList());
```

명령형 코드의 break, continue, return 등의 제어 흐름문을 모두 분석해서 같은 기능을 수행하는 스트림 연산을 유츄하는 것을 쉬운 일이 아니다.

하지만, 명령형 코드를 스트림 API로 바꾸도록 도움을 주는 도구들이 있다.

### 코드 유연성 개선

- 함수형 인터페이스 적용
  - 람다 표현식 이용 → 함수형 인터페이스가 필요
  - 조건부 연기 실행(conditional deferred execution)
    - 실제 작업을 처리하는 코드 내부에 제어 흐름문이 복잡하게 얽힌 경우
  - 실행 어라운드(execute around)
    - 매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드 → 람다로 변환
    - 준비, 종료 과정을 처리하는 로직을 재사용 → 코드 중복 줄임

## 람다로 객체지향 디자인 패턴 리팩토링

- 디자인 패턴
  - 다양한 패턴을 유형별로 정리한 것
  - 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공

### 전략 패턴 Strategy

전략 패턴 : 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법

예시) 오직 소문자 또는 숫자로 이루어져야 하는 등 텍스트 입력이 다양한 조건에 맞게 포맷되어 있는지 검증하는 경우

```java
public interface ValidatiaonStrategy {
		boolean execute(String s);
}

public class IsAllLowerCase implements ValidationStrategy {
		public boolean execute(String s) {
				return s.matches("[a-z]+");
		}
}

public class IsNumeric implements ValidataionStrategy {
		public boolean execute(String s) {
				return s.matches("\\d+");
		}
}

@RequiredArgsConstructor
public class Validator {
		private final ValidataionStrategy strategy;

		public boolean validate(String s) {
				return strategy.execute(s);
		}
}

Validator numericValidator = new Validator(new IsNumeric());
boolean b1 = numericValidator.validate("aaaa");
Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
boolean b2 = lowerCaseValidator.validate("bbbb");

// 람다 표현식 사용
Validator numericValidator = new Validator((String s) -> s.matches("[a-z]+"));
boolean b1 = numericValidator.validate("aaaa");
Validator lowerCaseValidator = new Validator((String s) -> s.matches("\\d+"));
boolean b2 = lowerCaseValidator.validate("bbbb");
```

람다 표현식으로 전략 디자인 패턴을 대신할 수 있다.

### 템플릿 메서드

알고리즘의 개요를 제시한 다음 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 템플릿 메서드 디자인 패턴을 사용한다.

```java
abstract class OnlineBanking {
		public void processCustomer(int id) {
				Customer c = Database.getCustomerWithId(id);
				makeCustomerHappy(c);
		}

		abstract void makeCustomerHappy(Customer c);
}
```

- 람다 표현식

  ```java
  public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
  		Customer c = Database.getCustomerWithId(id);
  		makeCustomer.accept(c);
  }

  new OnlineBankingLambda().processCustomer(1337, (Customer c) ->
  		System.out.println("Hello " + c.getName());
  ```

  이처럼 람다 표현식을 이용하면 템플릿 메서드 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있다.

### 옵저버

이벤트 발생 시 한 객체가 다른 객체 리스트에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다.

주로 GUI 애플리케이션에서 옵저버 패턴이 자주 등장한다.

```java
interface Observer {
		void notify(String tweet);
}

class NYTimes implements Observer {
		public void notify(String tweet) {
				if (tweet != null && tweet.contains("money")) {
						System.out.println("Breaking news in NY! " + tweet(;
				}
		}
}

class Guardian implements Observer {
		public void notify(String tweet) {
				if (tweet != null && tweet.contains("money")) {
						System.out.println("Yet another news in London... " + tweet(;
				}
		}
}
```

```java
interface Subject {
		void registerObserver(Observer o);
		void notifyObservers(String tweet);
}

class Feed implements Subject {
		private final List<Observer> observers = new ArrayList<>();

		public void registerObserver(Observer o) {
				this.observers.add(o);
		}

		public void notifyObservers(String tweet) {
				observers.forEach(o -> o.notify(tweet));
		}
}
```

```java
Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());

f.notifyObservers("The queen said her favourite book is Java 8 in Action!");
```

- 람다 표현식 사용하기

```java
f.registerObserver((String tweet) -> {
		if (tweet != null && tweet.contains("money")) {
				System.out.println("Breaking news in NY! " + tweet);
		}
});

f.registerObserver((String tweet) -> {
		if (tweet != null && tweet.contains("money")) {
				System.out.println("Yet another news in London... " + tweet);
		}
});
```

람다를 통해 불필요하게 감싸는 코드를 제거했다.

위 예제에서는 실행해야 할 동작이 비교적 간단하므로 람다 표현식으로 불필요한 코드를 제거하는 것이 좋다.

하지만, 옵저버가 상태를 가지며, 여러 메서드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현 방식을 고수하는 것이 바람직할 수도 있다.

### 의무 체인

작업처리 객체의 체인(동작 체인 등)을 만들 때는 의무 체인 패턴을 사용한다.

한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식이다.

일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다.

```java
public abstract class ProcessingObject<T> {
		protected ProcessingObject<T> successor;

		public void setSuccessor(ProcessingObject<T> successor) {
				this.successor = successor;
		}

		public T handle(T input) {
				T r = handleWork(input);
				if (successor != null) {
						return successor.handle(r);
				}
				return r;
		}

		abstract protected T handleWork(T input);
}
```

```java
public class HeaderTextProcessing extends ProcessingObject<String> {
		public String handleWork(String text) {
				return "From Raoul, Mario and Alan: " + text;
		}
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
		public String handleWork(String text) {
				return text.replaceAll("labda", "lambda");
		}
}

ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();

p1.setSuccessor(p2);

String result = p1.handle("Aren`t labdas really sexy?!!");
System.out.println(result);
```

- 람다 표현식 사용

의무 체인 패턴은 함수 체인과 비슷하다.

```java
UnaryOperator<String> headerProcessing =
		(String text) -> "From Raoul, Mario and Alan: " + text;

UnaryOperator<String> spellCheckerProcessing =
		(String text) -> text.replaceAll("labda", "lambda");

Function<String, String> pipeline =
		headerProcessing.andThen(spellCheckerProcessing);

String result = pipeline.apply("Aren`t labdas really sexy?!!");
```

### 팩토리

인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

```java
public class ProductFactory {
		public static Product createProduct(String name) {
				switch (name) {
						case "loan":
								return new Loan();
						case "stock":
								return new Stock();
						case "bond":
								return new Bond();
						default:
								throw new RuntimeException("No such product " + name);
				}
		}
}

Product p = ProductFactory.createProduct("loan");
```

- 람다 표현식 사용

```java
Supplier<Product> loanSupplier = Loan::new;
Loan loan = loanSupplier.get();

final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
		map.put("loan", Loan::new);
		map.put("stock", Stock::new);
		map.put("bond", Bond::new);
}

public static Product createProduct(String name) {
		Supplier<Product> p = map.get(name);
		if(p != null) return p.get();
		throw new IllegalArgumentException("No such product " + name);
}
```

다음과 같이 람다를 사용해 다양한 상품을 인스턴스화할 수 있다.

하지만, 팩토리 메서드 `createProduct` 가 생품 생성자로 여러 인수를 전달하는 상황에서는 이 기법을 적용하기 어렵다.

따라서, 상황에 맞게 사용해야 한다.

## 람다 테스팅

람다는 익명이므로 테스트 코드 이름을 호출할 수 없다.

- 람다를 테스트하는 방법

  - 보이는 람다 표현식의 동작 테스팅

    ```java
    @Test
    public void testMoveRightBy() throws Exception {
    		Point p1 = new Point(5, 5);
    		Point p2 = p1.moveRightBy(10);

    		assertEquals(15, p2.getX());
    		assertEquals(5, p2.getY());
    }
    ```

    ```java
    public final static Comparator<Point> compareByXAndThenY =
    		comparing(Point::getX).thenComparing(Point::getY);

    @Test
    public void testComparingTwoPoints() throws Exception {
    		Point p1 = new Point(10, 15);
    		Point p2 = new Potin(10, 20);
    		int result = Point.compareByXAndThenY.compare(p1, p2);
    		assertEquals(-1, result);
    }
    ```

  - 람다를 사용하는 메서드의 동작에 집중
    - 람다의 목표 : 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것
    - 람다를 사용하는 메서드를 테스트
  - 복잡한 람다를 개별 메서드로 분할
    - 람다 표현식을 메서드 레퍼런스로 바꾸기
  - 고차원 함수 테스팅
    - 메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트
    - 테스트해야 할 메서드가 다른 함수를 반환한다면 함수형 인터페이스의 인스턴스로 간주하고 함수의 동작을 테스트

## 디버깅

코드 디버깅 시 가장 먼저 확인해야 하는 두 가지

- 스택 트레이스
- 로깅

### 스텍 트레이스 확인

예외가 발생한다면 프로그램 실행이 어디서 멈췄고 어떻게 멈추게 되었는지 살펴봐야 한다.

> 스택 트레이스

프로그램이 멈췄다면 프로그램이 어떻게 멈추게 되었는지 프레임별로 보여주는 것

→ 문제가 발생한 지점에 이르게 된 메서드 호출 리스트를 얻을 수 있다.

람다의 경우 이름이 없기 때문에 조금 복잡한 스택 트레이스가 생성된다.

```java
points.stream().map(Point::getX).forEach(System.out::println);
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7245a6a6-1f15-4f93-962a-29adbd273eec/Untitled.png)

스택 트레이스로는 어디서 잘못되었는지 판단하기 힘들다.

메서드 레퍼런스를 사용해도 동일하다.

하지만, 메서드 레퍼런스를 사용하는 클래스와 같은 곳에 선언되어 있는 메서드를 참조할 때는 메서드 레퍼런스 이름이 스택 트레이스에 나타난다.

람다 표현식과 관련된 스택 트레이스는 이해하기 어렵다 → 미래의 자바 컴파일러가 개선해야 할 부분

### 정보 로깅

> peek

peek은 스트림 연산으로 스트림의 각 요소를 소비한 것처럼 동작을 실행한다.

하지만, forEach처럼 실제로 스트림의 요소를 소비하지는 않는다.

자신이 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달한다.

## Optional

NullPointerException으로부터 벗어나기 위한 하나의 방법

기존에는 if문을 사용하여 null 체크를 진행한다.

```java
if (a != null) {
	...
		if (b != null) {
				...
		}
}
```

위 방법은 if문으로 인해 코드의 들여쓰기가 증가한다.
이로 인해 코드의 구조가 엉망이 되고 가독성이 떨어진다.
이러한 반복 패턴 코드를 `깊은 의심 deep doubt` 이라고 한다.

```java
if (a == null) {
		return "Unknown";
}
...

if (b == null) {
		return "Unknown";
}
...
```

위와 같은 방법으로 중첩되는 if문을 없애고 들여쓰기 레벨을 줄일 수 있다.

하지만, 이 방법은 메서드의 출구가 여러 개 생기기 때문에 유지보수가 어려워진다.

### 자바에서 null 때문에 발생하는 문제들

- 에러의 근원 : NPE…
- 코드를 어지럽힘 : 중첩된 Null 확인 코드를 추가해야 함 → 가독성 하락
- 아무 의미가 없음
- 자바 철학에 위배 : 자바는 포인터를 숨겼다. 하지만, null은 포인터가 있다…
- 형식 시스템에 구멍을 만듦 : null은 무형식 & 정보 X → null이 있다면 무슨 의미로 사용되었는지 알 수 없음

### Optional 클래스

Optional<T> : 선택형값을 캡슐화하는 클래스

값이 없다면 Optional.empty()를 반환한다.

이로인해 NPE를 방지한다.

하지만, 모든 null을 Optional로 대치하는 것은 바람직하지 않다.

Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이다.

- Optional.empty() : 빈 Optional 객체
- Optional.of(T) : null이 아닌 값으로 만들어진 Optional 객체
- Optional.ofNullable(T) : null일수도 있는 값으로 만들어진 Optional 객체

  > Optional map

```java
String name = null;
if (insurance != null) {
		name = insurance.getName();
}
```

→

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

> Optional flatMap

```java
public String getCarInsuranceName(Optional<Person> person) {
		return person.flatMap(Person::getCar)
								 .flatMap(Car::getInsurance)
								 .map(Insurance::getName)
								 .orElse("Unknown");
}
```

> Optional 메서드

- get() : 값을 읽는 가장 간단한 메서드 & 동시에 가장 안전하지 않은 메서드
  - 래핑된 값이 있으면 해당 값을 반환, 없으면 `NoSuchElementException` 발생
  - 따라서, 중첩된 null 체크 코드를 넣는 것과 비슷한 상황 초래
- orElse(T other) : Optional이 값을 포함하지 않을 때 디폴트값을 제공 가능
- orElseGet(Supplier<? extends T> other) : orElse 메서드에 대응하는 게으른 버전
  - Optional에 값이 없을 때만 Supplier가 실행됨
  - 디폴트 메서드를 만드는 데 시간이 걸리거나 Optional이 비어있을 때만 디폴트값을 생성하고 싶다면 사용
- orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional이 비어있을 때 예외를 발생
  - 발생시킬 예외의 종류 선택 가능
- ifPresent(Consumer<? super T> consumer) : 값이 존재할 때 인수로 넘겨준 동작을 실행
  - 값이 없으면 아무 일도 일어나지 않음
    > 두 Optional 합치기

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
															Optional<Person> person, Optional<Car> car) {
		if (person.isPresent() && car.isPresent()) {
				return Optional.of(findCheapestInsurance(person.get(), car.get());
		} else {
				return Optional.empty();
		}
}
```

→

```java
public Optional<Insurance> nullSafeFindCheapsetInsurance(
															Optional<Person> person, Optional<Car> car) {
		return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

### 필터로 특정값 거르기

```java
Insurance insurance = ...;
if (insurance != null && "CambridgeInsurance".equals(insurance.getName())) {
		System.out.println("ok");
}
```

→

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance ->
													"CambridgeInsurance".equals(insurance.getName()))
						.ifPresent(x -> System.out.println("ok"));
```

Optional : 최대 한 개의 요소를 포함할 수 있는 스트림과 같다.

## Optional 응용

```java
Properties props = new Properties();
props.setProperty("a", "5");
props.setProperty("b", "true");
props.setProperty("c", "-3");

public int readDuration(Properties props, String name)
-> Properties를 읽어 값을 초 단위의 지속시간으로 해석

assertEquals(5, readDuration(param, "a"));
assertEquals(0, readDuration(param, "b"));
assertEquals(0, readDuration(param, "c"));
assertEquals(0, readDuration(param, "d"));
```

```java
public int readDuration(Properties props, String name) {
		String value = props.getProperty(name);
		if (value != null) {
				try {
						int i = Integer.parseInt(value);
						if (i > 0) {
								return i;
						}
				} catch (NumberFormatException nfe) { }
		}
		return 0;
}
```

→

```java
public int readDuration(Properties props, String name) {
		return Optional.ofNullable(props.getProperty(name))
									 .flatMap(OptionalUtility::stringToInt)
									 .filter(i -> i>0)
									 .orElse(0);
}
```

# CompletableFuture

## Future

- 자바 5부터 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있는 인터페이스
- 비동기 계산을 모델링
- 계산이 끝났을 때 결과에 접근할 수 있는 레퍼런스 제공

자바 8 이전 Future를 사용하는 예시 코드

```java
ExecutorService executor = Executors.newCachedThreadPool();
Future<Double> future = executor.submit(new Callable<Double>() {
		public Double call() {
				// 시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행
				return doSomeLongComputation();
		}
});

// 비동기 작업을 수행하는 동안 다른 작업을 수행
doSomethingElse();

try {
		Double result = future.get(1, TimeUnit.SECONDS);
} catch (ExecutionException ee) {
		...
}
...
```
