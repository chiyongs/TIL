## 소프트웨어 아키텍처

최근으로 넘어오면서 Antifragile이 주목되고 있다.

- Auto scaling : 사용량에 맞추어 서버를 증설하거나 감축시킨다.
- Microservices : Cloud native architecture, application에 중요한 개념, 모든 서비스들이 독립적으로 운영할 수 있도록 분리된 시스템
- Chaos engineering : 시스템이 예측하기 못한 상황도 견뎌낼 수 있도록 구축되어야 한다.
- Continuous deployments : CI/CD 배포 라인, Cloud native 서비스는 MSA 구조로 서비스가 분리되어 있기 때문에 각각의 서비스를 빌드하고 배포하는 시스템을 자동화

### Cloud Native Architecture

- 확장 가능한 아키텍처
  - 시스템의 수평적 확장에 유연
  - 확장된 서버로 시스템의 부하 분산, 가용성 보장
  - 시스템 또는 서비스 애플리케이션 단위의 패키지 (컨테이너 기반 패키지)
  - 모니터링
- 탄력적 아키텍처
  - 서비스 생성-통합-배포, 비즈니스 환경 변화에 대응 시간 단축
  - 분할된 서비스 구조
  - 무상태 통신 프로토콜
  - 서비스의 추가와 삭제를 자동으로 감지
  - 변경된 서비스 요청에 따라 사용자 요청 처리
- 장애 격리
  - 특정 서비스에 오류가 발생해도 다른 서비스에 영향을 주지 않음

### 12 Factors

Cloud native application을 만들기 위해 고려해야 하는 12가지 조건

- 코드 통합
  - 버전을 제어하기 위한 목적
  - 형상 관리를 위해서 코드를 한 곳에서 배포하는 것이 주 목적
- 종속성의 배제
- 환경설정의 외부관리
  - 구성정보, 코드 외부에서 구성 관리 도구를 통해서 아키텍처에 필요한 작업들을 주입
- 백업 서비스의 분리
- 개발환경과 테스트,운영환경의 분리
- 상태 관리
- 포트 바인딩
- 동시성
- 서비스의 올바른 상태 유지
- 개발과 운영 환경의 통일
- 로그의 분리
- 관리 프로세스

### Monolithic vs MSA

Monolithic(모놀리식) 아키텍처

- 모든 업무 로직이 하나의 애플리케이션 형태로 패키지되어 서비스
- 애플리케이션에서 사용하는 데이터가 한 곳에 모여 참조되어 서비스되는 형태

Microservice 아키텍처

- 함께 돌아가는 작고 자동화되어있는 서비스들의 모임
- MSA는 작은 규모에 맞는 하나의 애플리케이션을 개발하는 것에서부터 시작되었다.
- 이러한 작은 단위 서비스들이 모여 각각의 컨테이너가 되고 이들이 모여 큰 조직이 된다.

### SOA와 MSA

- SOA(Service Oriented Architecture)
  - 비즈니스 측면에서 서비스 재사용성
  - ESB(Enterprise Service Bus)라는 서비스 채널을 이용하여 서비스를 공유하고 재사용한다.
- MSA(Microservice Architecture)
  - 한 가지 작은 서비스에 집중
  - 서비스를 공유하지 않고 독립적으로 실행한다.

차이점

- 서비스 공유에 대한 지향점이 다르다
- SOA : 서비스 공유를 최대한 많이 하며 재사용을 통한 비용 절감 고려
- MSA : 서비스 공유를 최소화하여 서비스 간의 결합도를 낮추어 변화에 능동적으로 대응

기술방식

- SOA : 공통의 서비스를 ESB에 모아 사업 측면에서 공통 서비스 형식으로 서비스 제공
- MSA : 각 독립된 서비스가 노출된 REST API를 사용

### Spring Cloud

- Spring Cloud Config Server : 중앙 통합 환경 설정 관리
- Eureka(Naming Server)
- Spring Cloud Gateway & Ribbon : 로드 밸런싱
- FeignClient : Easier REST Clients
- Zipkin Distributed Tracing, Netflix API gateway : 모니터링
- Hystrix : Fault Tolerance

## Service Discovery

Eureka : MSA 안에서 각각의 서비스들이 다른 서비스들을 찾기 위한 전화번호부같은 역할

먼저, 서버 인스턴스를 Spring Cloud Netflix Eureka에 등록해야 한다.

`@EnableEurekaServer` : EurekaServer의 작업을 수행하도록 하는 어노테이션

```yaml
server:
  port: 8761

spring:
  application:
    name: discoveryservice

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

`eureka.client.register-with-eureka,fetch-register` : Eureka 라이브러리가 포함된채 Spring Boot가 실행하게 되면 위 2가지 옵션이 true로 설정되어 Eureka Client도 수행하려한다. 하지만, 우리의 경우 Server의 역할만 담당하므로 false로 설정한다.
