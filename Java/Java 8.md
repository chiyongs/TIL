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
