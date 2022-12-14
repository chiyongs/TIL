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

### 주문 조회 V3 : 엔티티를 DTO로 변환 - 페치 조인 최적화

1대다 관계(컬렉션)를 조인하게 되면 데이터베이스 내에서 결과가 뻥튀기가 된다.

```java
public List<Order> findAllWithItem() {
    return em.createQuery(
            "select o from Order o" +
                    " join fetch o.member m" +
                    " join fetch o.delivery d" +
                    " join fetch o.orderItems oi" +
                    " join fetch oi.item i",Order.class)
            .getResultList();
}
```

따라서, 위와 같이 페치 조인을 사용하게 되면 중복된 부풀려진 데이터가 결과로 반환된다.

또한, 반환된 데이터들 중 id가 같은 데이터들은 reference 주소까지 동일하다.

JPA에서는 id값이 같으면 같은 데이터로 처리하기 때문이다.

그렇다면 어떻게 해결해야 할까?

여기서 `distinct` 키워드를 사용할 수 있다.

```java
public List<Order> findAllWithItem() {
    return em.createQuery(
            "select distinct o from Order o" +
                    " join fetch o.member m" +
                    " join fetch o.delivery d" +
                    " join fetch o.orderItems oi" +
                    " join fetch oi.item i",Order.class)
            .getResultList();
}
```

SQL에서 사용되는 `distinct` 키워드는 row 내의 모든 데이터가 동일해야만 중복을 제거한다.

JPA의 `distinct` 키워드는 두 가지 기능을 제공한다.

- DB SQL에 distinct 키워드를 날려주고,
- 애플리케이션 단에서 Root Entity가 중복인 경우에 걸러서 컬렉션에 담아준다.

이로써 SQL이 1번만 실행된다.

컬렉션 페치 조인의 단점

- 페이징

1대다 관계에서 페치 조인을 사용하게 되면 페이징이 불가능하다.

JPA의 구현체인 하이버네이트는 경고 로그를 남기면서 모든 데이터를 DB에서 불러와 메모리에서 페이징을 진행한다. → 매우 위험 (Out of Memory 위험)

또한, 컬렉션(1대다) 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 데이터가 부정합하게 조회될 수 있다.

### 주문 조회 V3.1 : 엔티티를 DTO로 변환 - 페이징과 한계 돌파

컬렉션을 페치 조인하면 페이징이 불가능하다.

이유 : 일대다 관계를 조인한다면 데이터가 예측할 수 없이 증가하기 때문이다. 하지만, 데이터는 다(N)을 기준으로 row가 생성되기 때문이다.

1. ToOne(OneToOne, ManyToOne) 관계는 모두 페치 조인을 사용한다.

   ```java
   public List<Order> findAllWithMemberDelivery(int offset, int limit) {
       return em.createQuery(
               "select o from Order o" +
                       " join fetch o.member m" +
                       " join fetch o.delivery d",Order.class)
               .setFirstResult(offset)
               .setMaxResults(limit)
               .getResultList();
   }
   ```

2. 컬렉션은 지연 로딩을 사용한다.
3. 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`, `@BatchSize` 를 적용한다.
   1. `default_batch_fetch_size: 100`

`hibernate.default_batch_fetch_size` 를 사용하면 기존 N+1 문제가 발생하던 부분이 SQL에서 where-in 쿼리를 통해 한번에 데이터를 가져오게 된다.

이를 통해 쿼리 수를 매우 줄일 수 있다.

V3 : 쿼리는 더 적게 나가지만 애플리케이션으로 전송되는 데이터의 용량은 더 많다.

V3.1 : 쿼리는 좀 더 나가지만 애플리케이션으로 전송되는 데이터의 용량이 더 적다.

`default_batch_fetch_size` 는 100~1000 사이의 적당한 사이즈를 선택해야 한다.

1000으로 잡으면 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB에 순간 부하가 증가할 수 있으므로 적당한 값을 잘 설정해야 한다.

### 주문 조회 V4 : JPA에서 DTO 직접 조회

JPA에서 바로 DTO에 값을 채워 조회하는 메서드는 화면과 직접적으로 연관된 메서드인 경우가 많다.

따라서, 일반적으로 사용하는 리포지토리에 메서드를 만들지않고 따로 분리하여 만든다.

이유 : 화면과 연관된 메서드와 비즈니스 로직과 연관된 메서드의 라이프사이클이 다르기 때문

DTO를 바로 조회하여도 루트 쿼리 1번에 컬렉션을 조회하기 위한 N번의 쿼리가 발생하게 된다.

```java
public List<OrderQueryDto> findOrderQueryDtos() {
    List<OrderQueryDto> result = findOrders();
    result.forEach(o-> {
        List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
        o.setOrderItems(orderItems);
    });
    return result;
}

private List<OrderItemQueryDto> findOrderItems(Long orderId) {
    return em.createQuery(
            "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                    " from OrderItem oi" +
                    " join oi.item i" +
                    " where oi.order.id =:orderId",OrderItemQueryDto.class)
            .setParameter("orderId",orderId)
            .getResultList();
}

private List<OrderQueryDto> findOrders() {
    return em.createQuery(
                    "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                            " from Order o" +
                            " join o.member m" +
                            " join o.delivery d",OrderQueryDto.class)
            .getResultList();
}
```

### 주문 조회 V5 : JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

컬렉션을 조회하기 위해 N번의 쿼리가 발생하던 것을 SQL의 where-in을 사용하여 1번의 쿼리로 한번에 다 가져온다.

이를 통해 V4에 비해 쿼리가 적게 나간다.

또한 Map을 사용해서 매칭 성능을 최적화했다. 시간 복잡도 O(1)### 주문 조회 V5 : JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

컬렉션을 조회하기 위해 N번의 쿼리가 발생하던 것을 SQL의 where-in을 사용하여 1번의 쿼리로 한번에 다 가져온다.

이를 통해 V4에 비해 쿼리가 적게 나간다.

또한 Map을 사용해서 매칭 성능을 최적화했다. 시간 복잡도 O(1)

```java
public List<OrderQueryDto> findAllByDto_optimization() {
    List<OrderQueryDto> result = findOrders();

    Map<Long,List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(toOrderIds(result));

    result.forEach(o->o.setOrderItems(orderItemMap.get(o.getOrderId())));

    return result;
}

private Map<Long,List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
    List<OrderItemQueryDto> orderItems = em.createQuery(
                    "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                            " from OrderItem oi" +
                            " join oi.item i" +
                            " where oi.order.id in:orderIds",OrderItemQueryDto.class)
            .setParameter("orderIds",orderIds)
            .getResultList();

    Map<Long,List<OrderItemQueryDto>> orderItemMap = orderItems.stream()
            .collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
    return orderItemMap;
}
```

### 주문 조회 V6 : JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

모든 테이블을 다 조인해서 한 방 쿼리로 모든 데이터를 가져온다.

이러면 쿼리가 한번만 발생한다는 장점이 있지만, 페이징이 불가능하다는 단점을 가지고 있다.

우리는 `OrderQueryDto` 라는 API 스펙으로 반환하려 했다.

따라서, 변환 작업이 필요하다.

stream을 사용해서 메모리상에서 변환작업을 진행한다.

```java
@GetMapping("/api/v6/orders")
public List<OrderQueryDto> ordersV6() {
    List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

    return flats.stream()
            .collect(groupingBy(o-> new OrderQueryDto(o.getOrderId(),o.getName(),o.getOrderDate(),o.getOrderStatus(),o.getAddress()),
mapping(o-> new OrderItemQueryDto(o.getOrderId(),o.getItemName(),o.getOrderPrice(),o.getCount()),toList())
            )).entrySet().stream()
            .map(e-> new OrderQueryDto(e.getKey().getOrderId(),e.getKey().getName(),e.getKey().getOrderDate(),e.getKey().getOrderStatus(),e.getKey().getAddress(),e.getValue()))
            .collect(toList());
}
```

하지만, 위와 같이 만들어도 우리가 원하는 중복되지 않은 데이터가 나오지 않고 조인 결과가 나오게 된다.

이를 해결하기 위해서 `OrderQueryDto` 에 _`@EqualsAndHashCode_(of = "orderId")` 인 Lombok 어노테이션을 통해 orderId가 같을 시 같은 객체로 인식시켜주는 작업이 추가로 필요하다.

> V6 정리

쿼리는 한번이지만 조인으로 인해 중복데이터가 추가되므로 상황에 따라 더 느릴 수도 있다.

또한, 애플리케이션에서 추가 작업이 크고 페이징이 불가능하다.
