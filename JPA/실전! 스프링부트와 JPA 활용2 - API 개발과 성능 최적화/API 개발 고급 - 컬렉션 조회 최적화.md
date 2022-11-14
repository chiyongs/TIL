일대다 관계에서는 조인을 하게 되면 데이터가 뻥튀기가 되어 최적화하기가 어려워진다.

### 주문 조회 V1 : 엔티티 직접 노출

```java
@GetMapping("/api/v1/orders")
public List<Order> ordersV1() {
List<Order> all = orderRepository.findAllByString(new OrderSearch());
    for (Orderorder : all) {
        order.getMember().getName();
        order.getDelivery().getAddress();

        List<OrderItem> orderItems = order.getOrderItems();
        orderItems.stream().forEach(o->o.getItem().getName());
    }
    return all;
}
```

Entity를 직접 노츨할 때 양방향 연관관계에 대해서는 무한 루프가 걸리지 않게 무조건 한 곳에 `@JsonIgnore` 를 추가해야 한다.

Entity를 직접 노출하는 것은 좋은 방법이 아니다.

### 주문 조회 V2 : 엔티티를 DTO로 변환

```java
@GetMapping("/api/v2/orders")
public List<OrderDto> ordersV2() {
List<Order> orders = orderRepository.findAllByString(new OrderSearch());
List<OrderDto> result = orders.stream()
            .map(OrderDto::new)
            .collect(toList());

    return result;
}
```

```java
@Getter
static class OrderDto{

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDto> orderItems;

    public OrderDto(Orderorder) {
        this.orderId = order.getId();
        this.name = order.getMember().getName();
        this.orderDate = order.getOrderDate();
        this.orderStatus = order.getStatus();
        this.address = order.getDelivery().getAddress();
        this.orderItems = order.getOrderItems().stream()
                .map(OrderItemDto::new)
                .collect(toList());
    }
}

@Getter
static class OrderItemDto{

    private String itemName;
    private int orderPrice;
    private int count;
    public OrderItemDto(OrderItemorderItem) {
        itemName = orderItem.getItem().getName();
        orderPrice = orderItem.getOrderPrice();
        count = orderItem.getCount();
    }
}
```

DTO로 변환했을 때 또 그 안에 Entity가 들어있다면 해당 Entity도 DTO로 변환해주어야 한다.

왜냐하면 이전에도 그랬듯이 Entity가 API에 직접 노출된다면 서로의 변경에 취약해지기 때문이다.

하지만, V2도 실행해보면 컬렉션이라는 점과 지연 로딩때문에 엄청 많은 양의 쿼리가 발생한다.
