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

### 간단한 주문 조회 V2 : 엔티티를 DTO로 변환

```java
@Data
static classSimpleOrderDto{
    privateLongorderId;
    privateStringname;
    privateLocalDateTimeorderDate;
    privateOrderStatusorderStatus;
    privateAddressaddress;

    public SimpleOrderDto(Orderorder) {
        this.orderId =order.getId();
        this.name =order.getMember().getName();// LAZY초기화
        this.orderDate =order.getOrderDate();
        this.orderStatus =order.getStatus();
        this.address =order.getDelivery().getAddress();// LAZY초기화
    }
}
```

```java
@GetMapping("/api/v2/simple-orders")
public List<SimpleOrderDto> ordersV2() {
    // ORDER 2개
    // N + 1 -> 1 +회원 N +배송 N
    List<Order> orders = orderRepository.findAllByString(new OrderSearch());

    List<SimpleOrderDto> result = orders.stream()
            .map(SimpleOrderDto::new)
            .collect(Collectors.toList());

    return result;
}
```

위 경우에서는 Order, Member, Delivery, 총 3개의 테이블을 참조하게 된다.

`orderRepository.findAllByString()` 을 통해 2개의 Order를 발견하게 되고,

이를 stream 돌면서 각 Order에 필요한 Member, Delivery를 그때그때(`LAZY`) 채워넣게 된다.

따라서, 쿼리가 너무 많이 나가게 된다. (N+1 문제)

너무 많은 쿼리는 성능 상 좋지 않다.

그렇다면 fetchType을 `EAGER` 로 바꾸게 되면 어떻게 될까?

`EAGER` 로 바꾸고 실행한다면 한 번에 모든 것을 가져오려고 하다보니 이 또한 많은 쿼리와 성능이 좋지 못한 쿼리가 발생한다.

또한, `EAGER` 를 사용한다면 쿼리를 예측하기 어려워진다.

따라서, 이런 문제를 해결하기 위해서는 `Fetch Join`을 사용해야 한다!!!

### 간단한 주문 조회 V3 : 엔티티를 DTO로 변환 - 페치 조인 최적화

실무의 대부분 성능 문제는 N+1 문제로부터 발생한다.

`fetch join`은 필요한 객체 그래프를 한번에 다 쿼리해올때 사용한다.

`fetch join`을 제대로 이해하고 사용하면 90%의 성능 문제는 해결할 수 있다.

```java
@GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDto> ordersV3() {
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    List<SimpleOrderDto> result = orders.stream()
            .map(SimpleOrderDto::new)
            .collect(Collectors.toList());

    return result;
}
```

페치 조인을 통해 V2에서 발생하던 N+1 문제(필요한 Member와 Delivery를 Lazy 로딩으로 인해 계속 쿼리가 나가는 문제)를 해결!!

### 간단한 주문 조회 V4 : JPA에서 DTO로 바로 조회

```java
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDto> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();
}
```

```java
@Data
public class OrderSimpleQueryDto{
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId,String name,LocalDateTime orderDate,OrderStatus orderStatus,Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

V3 페치 조인을 사용할 때는 페치 조인한 모든 Entity들의 정보를 다 가져왔다.

지금 V4는 필요한 데이터만 쏙쏙 가져온다.

이로써 애플리케이션 네트워크 용량 최적화가 된다. (하지만, 생각보다 미비)

하지만, API 스펙에 맞게 데이터를 쿼리하는 것은 API 스펙 또는 화면에 의해 리포지토리를 변경해야 하므로 순수한 Entity를 조회하거나 객체 그래프를 조회하는 리포지토리의 목적에 벗어난다.

따라서, 이같은 API 스펙에 최적화된 용도가 필요할 땐 리포지토리 하위에 별도의 패키지를 만든다.

실무에서는 많이 복잡한 쿼리와 DTO를 뽑아야할 때가 많은데 이때의 유지보수성을 위해 분리한다.

Entity를 조회하고 DTO로 변환하는 방법과 바로 DTO를 조회하는 방법, 두 가지 모두 트레이드오프가 있다.

따라서, 쿼리 방색 선택 권장 순서는 다음과 같다.

1. 우선 Entity를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화한다. → 대부분의 성능 이슈가 해결된다.
3. 그래도 해결 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template를 사용해서 SQL을 직접 사용한다.
