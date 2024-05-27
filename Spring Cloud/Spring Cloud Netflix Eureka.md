# Spring Cloud Netflix Eureka

Spring Cloud Netflix Eureka는 대상과 대상의 전화번호 등의 정보를 가지고 있는 전화번호부와 비슷하다.
API Gateway로부터 들어온 요청을 어느 서비스로 전달해야 하는지 파악하고 알려주는 역할

> Service Registration

MSA 인스턴스가 시작될 때, host와 port, health check URL, 홈페이지 등의 메타데이터와 함께 Eureka 서버에 자신을 등록

> Service Discovery

MSA 환경에서 Eureka 서버를 통해 서비스들이 서로를 찾을 수 있음
클라이언트는 서버에서 Registry 정보를 가져와 로컬에 캐시
이 정보는 주기적으로 서버에서 업데이트됨

> Load Balancing

Eureka 내부에는 여러 서비스 인스턴스들에 요청을 고르게 분배하는 내장형 로드 밸런서가 포함
트래픽을 직접 처리하지 않지만, Ribbon과 같은 클라이언트 측 로드 밸런서가 요청을 효율적으로 라우팅할 수 있도록 필요한 메타데이터를 제공

> Resilience

네트워크 문제가 있을 때 등록 및 쿼리를 받아들일 수 있도록 회복 탄력성을 고려하여 설계
클라이언트는 또한 서비스 레지스트리 정보를 로컬에 캐시하여 Eureka 서버가 일시적으로 실패하더라도 캐시된 데이터를 사용하여 계속 작동할 수 있음

> Self-Preservation

통신 네트워크 문제로 인해 마이크로서비스와 Eureka 서버 간에 레지스트리가 빠르게 인스턴스를 제거하는 것을 방지하는데 도움
서비스 등록 및 디스커버리가 동적으로 필요한 아키텍처에 잘 맞으며, 특히 자동 스케일링 및 기타 요인으로 인해 호스트 주소와 포트 번호가 자주 변경될 수 있는 클라우드 환경에서 유용

MSA에서 서비스 간 통신 관리를 단순화하며, MSA의 개발, 배포, 확장을 독립적으로 쉽게 할 수 있도록 도움