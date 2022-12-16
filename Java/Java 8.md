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
