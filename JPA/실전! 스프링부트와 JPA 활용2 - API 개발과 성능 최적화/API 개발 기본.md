### 강의 서론

JPA의 성능 최적화의 방법을 잘 모르는 분들이 많다.

JPA는 그 자체로 수많은 성능 튜닝을 제공한다.

JPA를 사용하면서 어떻게 API를 설계하는게 올바른 방법인지에 대한 강의!

### 회원 등록 API

개발할 때 패키지별로 예외처리하는 경우가 많이 존재한다.

따라서, API controller와 화면을 담당하는 controller를 분리하는 것이 좋다.

- RequestBody로 Member Entity를 바로 받아서 처리하는 API

```java
@PostMapping("/api/v1/members")
public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

위처럼, Entity를 RequestBody로 받게 되면 Entity 자체에 API 검증을 위한 로직이 들어가야 하며, Entity에 변경이 생기면 API 스펙이 변하게 되는 등 다양한 문제점이 존재한다.

주의 🔥

Entity는 파라미터로 받지도 말아야 하고 외부에 노출해서도 안된다.

이유 : Entity는 공통적으로 많이 사용하기 때문에 다양한 요구사항을 충족할 수 없을 뿐더러 변경 시 위험하기 때문이다.

- Entity 대신 DTO를 RequestBody에 매핑

```java
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
    Member member = new Member();
    member.setName(request.getName());

    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

이처럼 DTO를 사용하게 되면 API SPEC이 변경되어도 Entity에 변경이 발생하지 않는다.

또한, DTO를 사용하는 것이 확인하기도 명확하며 편리하다.

### 회원 조회 API

Entity를 직접 노출하게 되면 Entity에 있는 모든 정보들이 다 노출이 된다.

`@JsonIgnore` 어노테이션으로 막을 순 있지만 다른 문제점들이 많다.

```java
@GetMapping("/api/v1/members")
public List<Member> membersV1() {
    return memberService.findMembers();
}
```

- Entity로는 다양한 API SPEC에 맞출 수 없다.
- Entity에 표현 계층을 위한 로직이 포함되어 있는 것은 좋지 않다.
- Entity 변경 시 API SPEC이 변경된다.
- 컬렉션을 직접 반환하면 향후 API SPEC을 변경하기 어렵다. → 별도의 Result 클래스를 생성하여 해결해야 함

따라서, 별도의 DTO로 변환하여 API 응답을 해야 한다!!

```java
@GetMapping("/api/v2/members")
public Result<?> memberV2() {
    List<Member> findMembers = memberService.findMembers();
    List<MemberDto> collect = findMembers.stream()
            .map(m -> new MemberDto(m.getName()))
            .collect(Collectors.toList());

    return new Result(collect);
}

@Data
@AllArgsConstructor
static class Result<T> {
    private T data;
}

@Data
@AllArgsConstructor
static class MemberDto {
    private String name;
}
```

Entity를 DTO로 변환하는 수고로움이 생기지만, API SPEC과 Entity는 서로의 변경에 영향이 없어진다.

API를 만들 때에는 절대 Entity를 노출시키지 않고 무조건 DTO를 사용해야 한다.