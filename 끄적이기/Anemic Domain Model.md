1. Anemic Domain Model이란?
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
