## 1. Anemic Domain Model이란?

Anemic Domain Model(빈약한 도메인 모델)은 **객체 지향 설계의 반패턴(anti-pattern)**으로, 도메인 객체가 본래 가져야 할 비즈니스 로직이나 행동(behavior)을 포함하지 않고, 단순히 데이터를 담는 컨테이너로만 사용되는 설계를 말합니다.

주요 특징:
객체에 비즈니스 로직이 없음.
Getter/Setter 메서드만 포함된 데이터 구조체와 유사함.
비즈니스 로직은 서비스 계층(Service Layer)에 집중됨.
예를 들어, 아래와 같은 코드가 대표적입니다.

```java
class Order {
    private String orderId;
    private double totalAmount;

    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public double getTotalAmount() {
        return totalAmount;
    }

    public void setTotalAmount(double totalAmount) {
        this.totalAmount = totalAmount;
    }
}
```

비즈니스 로직은 객체가 아닌 별도의 서비스에서 처리됩니다.

```java
class OrderService {
    public double calculateDiscount(Order order) {
        if (order.getTotalAmount() > 100) {
            return order.getTotalAmount() * 0.1;
        }
        return 0;
    }
}

```

## 2. 문제점

객체 지향 프로그래밍(OOP)의 원칙 위반

- OOP는 **데이터(data)**와 그 데이터를 다루는 **행동(behavior)**를 객체 내부에 캡슐화하는 것을 목표로 합니다.
- Anemic Domain Model은 데이터와 행동이 분리되어 절차적 프로그래밍과 유사해지며, 객체 지향의 장점을 잃습니다.

높은 결합도와 낮은 응집도

- 비즈니스 로직이 서비스 계층에 집중되면, 서비스 클래스가 커지고 복잡해집니다.
- 데이터와 로직이 분리되어 코드 변경 시 서로 간의 의존성이 높아집니다.
  유지보수 어려움
- 비즈니스 로직이 여러 서비스 클래스에 분산될 수 있어, 유지보수가 어려워집니다.
- 로직이 도메인 객체와 함께 캡슐화되지 않으므로 코드의 의미를 이해하기 힘들어집니다.

## 3. 풍부한 도메인 모델(Rich Domain Model)

Fowler는 Anemic Domain Model의 대안으로 Rich Domain Model을 제안합니다. 이는 객체가 데이터와 행동을 함께 가지는 모델입니다.

Rich Domain Model의 특징:
객체 내부에 비즈니스 로직 포함.
캡슐화(encapsulation)와 객체 간 책임 분담을 강화.
객체 스스로가 자신의 상태와 행동을 관리.
예시:

```java
class Order {
    private String orderId;
    private double totalAmount;

    public Order(String orderId, double totalAmount) {
        this.orderId = orderId;
        this.totalAmount = totalAmount;
    }

    public double calculateDiscount() {
        if (this.totalAmount > 100) {
            return this.totalAmount * 0.1;
        }
        return 0;
    }
}
```

여기서는 Order 객체가 자신이 가진 데이터를 기반으로 할 수 있는 행동(calculateDiscount)을 스스로 처리합니다.

## 4. Rich Domain Model의 장점

1. 캡슐화 강화
   객체가 자신의 데이터를 보호하며, 로직과 데이터를 한곳에 모아 응집도가 높아집니다.

2. 서비스 계층 간소화
   서비스 계층이 비즈니스 로직을 직접 처리하지 않고, 도메인 객체에 위임합니다. 이를 통해 서비스 클래스는 더 작고 간단해집니다.

3. 코드의 가독성과 유지보수성 향상
   로직이 객체 내부에 위치하므로, 도메인 객체를 읽는 것만으로 비즈니스 로직을 이해할 수 있습니다. 로직이 분산되지 않아 코드 탐색이 쉬워집니다.
