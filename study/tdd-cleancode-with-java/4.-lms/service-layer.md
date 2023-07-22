# Service Layer

## Layered Architecture(계층구조)

layered Architecture는 코드를 모듈화하고 재사용성을 높여서, 애플리케이션을 쉽게 확장하고 유지보수 하기 위해 사용한다. 즉, 각 계층이 독립적으로 개발, 테스트될 수 있으며, 따라서 애플리케이션 전체의 유지보수성이 향상된다.

각 계층은 분명 서로 상호작용이 일어나지만, 다른 계층에서 발생하는 일에 대해선 전혀 몰라도 된다.

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

* Presentation Layer
  * Spring MVC패턴을 통해 사용자의 요청을 처리하고 응답 생서
* Persistence Layer
  * 데이터 영속성을 관리하고 저장, 검색, 수정, 삭제 등의 작업을 수행

## Service Layer의 역할

Data Access Layer는 개별  SQL의 처리를 목표로 한다. 즉 여러 SQL을 묶어서 사용하지 않는다. 하지만 업무는 여러개의 SQL들이 뭉쳐서 동작한다.&#x20;

특정 업무에서 하나의 단위를 일이 진행되야하는 경우가 있다. 이때 필요한 것이 Transaction의 개념이다. DAO는 개별 SQL을 처리하면 끝나버리기 때문에 업무에 역인 다른 SQL을 살펴볼 수가 없다. 따라서 여러 Data Access Layer들의 동작을 묶어서 하나로 관리할 개념이 필요해졌고 이것이 Service Layer이다.&#x20;

## Service Layer의 필요성

* 비지니스 로직의 분리와 중앙화
  * 도메인 객체간의 상호작용
  * Persistence Layer와 데이터를 주고받으면 결과를 Controller에게 반환
* 각 계층의 역할 명확화
  * Controller는 사용자의 요청을 처리하고 응답을 생성
  * Persistence Layer는 DB와의 상호작용을
  * Service Layer는 비즈니스 로직과 트랜잭션 관리 수행
* 트랜잭션 관리
  * Contoller에서 관리하기엔, 트랜잭션의 단위가 너무 커
  * 비즈니스 로직 처리와 트랜잭션 처리가 동시에 처리하기 위해 여러 Dao 혹은 Service를 통해 데이터를 다뤄야 할 때, 트랜잭션의 경계를 더 명확히 할 수 있다.
  * Persistence Layer에서 트랜잭션을 관리하기엔, 비즈니스 로직과 데이터의 영속성과 관련된 작업이 혼재될 수 있다.

{% embed url="https://tjdtls690.github.io/studycontents/java/2023-04-24-service_layer/" %}

{% embed url="https://goodteacher.tistory.com/252" %}
