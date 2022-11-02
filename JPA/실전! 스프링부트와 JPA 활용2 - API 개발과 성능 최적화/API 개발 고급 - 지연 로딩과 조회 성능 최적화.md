### 간단한 주문 조회 V1 : 엔티티를 직접 노출

```java
@GetMapping("/api/v1/simple-orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    return all;
}
```

위와 같이 API를 만들게 되면 Order와 양방향 연관관계가 걸려있는 Entity들로 인해 무한 루프가 발생한다.

따라서, 양방향 연관관계로 만들어진 무한루프를 해결하기 위해서는 `@JsonIgnore` 를 통해 한 쪽 Entity를 Json 변환을 금지해야 한다.

하지만, 이렇게 하여도 오류가 발생한다.

왜냐하면, `fetchType=Lazy` 로 설정해놓았기 때문에 `ByteBuddy` 라이브러리를 사용하여 만들어진 Proxy객체들이 들어가 있기 때문에 jackson 라이브러리가 이를 parsing하지 못하기 때문이다.

이를 해결하기 위해서는 `Hibernate5Module` 이라는 것을 사용하여 가능하지만,

애초에 Entity를 직접 노출하는 것이 문제이므로 시도할 필요가 없다.

```java
@GetMapping("/api/v1/simple-orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Order order : all) {
        order.getMember().getName();// LAZY 강제 초기화
        order.getDelivery().getAddress();// LAZY 강제 초기화
    }
    return all;
}
```

위와 같이 반복문을 통해 강제로 LAZY 로딩을 초기화하여 값을 채워넣을 수 있다.

하지만, 이조차도 쓸모없는 데이터가 너무 많다.

API Spec에는 꼭 필요한 데이터만 들어있어야 한다.
