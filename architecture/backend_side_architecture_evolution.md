> ###

> <a href="https://medium.com/@iamprovidence/backend-side-architecture-evolution-n-layered-ddd-hexagon-onion-clean-architecture-643d72444ce4">백엔드 아키텍쳐 진화</a>

GoF

- 요구사항이 복잡해지면서, UI를 비즈니스 로직과 분리하며 MVC 패턴이 등장함.
  - M은 Model을 의미하고, repository, service, entity를 포함한 비즈니스 로직이 포함되어있다. 따라서, 모든 복잡한 코드들이 Model에 모여있다는 단점이 있다.
  - GoF에서 MVC로 변경된 것은 로직을 분리하는 것으로부터 시작했고 이는 Divide and Concquer 방식이다. MVC를 추가로 Divde and Conquer 해보자.

## 2002 N Layered

- 마틴 파울러는 Patterns of Enterprise Application Architecture라는 책에서 N-layered 구조를 제안했다. 이는 관련된 코드를 묶어 하나의 레이어로 정의하는 방식이다.

- 다음은 N layered에서 일관성을 유지하기 위한 규칙이다.

  - 레이어 네이밍은 자유롭게
  - 필요한 만큼 레이어를 가질 수 있다
  - 레이어 사이에 레이어를 추가할 수 있다
  - 하나의 레이어에 여러개의 컴포넌트를 가질 수 있다
  - 레이어 간 명확한 위계가 존재하고, 서로를 참조할 수 있도록 한다
  - 이는 코드 중복을 없애고 코드를 구조화할 수 있게 되었다.

- 대부분 User Interface, Business Logic, Data Access 이렇게 3가지 레이어로 나눈다.

  - User Interface (UI): 유저 인터랙션과 관련된 부분
  - Business logic layer (BLL): 비즈니스 로직을 담고 있으며, 애플리케이션의 핵심 코드들을 담고 있다.
  - Data Access layer (DAL): 메모리에 데이터를 영속화하고, 애플리케이션의 상태를 유지한다.

- User Interface -> Business Logic -> Data Access (의존성 방향)

- 레이어 간 분리는 폴더 구조로 나눌 수도 있고 프로젝트로 나눌 수도 있다.

  프로젝트로 분리한 경우, 폴더구조와는 다르게 의존성(dependency)를 유의미하게 제어할 수 있다. 하지만 프로젝트가 너무 많아질 경우, 유지보수에 어려울 수 있다. 현재 상황에 적합한 구조를 따르는 것이 적절하다.

## 2003 Domain Driven Design

- 에릭 에반스는 Domain-Driven Design: Tackling Complexity in the Heart of Software라는 책을 출판했다.

- 에릭 에반스는 프로젝트의 의존성이 한 방향이어야 한다는 파울러의 의견에 동의했다. 하지만, 그는 종속성 방향 규칙을 위반하지 않는 한 저수준 모듈이 고수준 모듈을 호출해도 괜찮다는 의견을 제시했다.

- DDD는 레이어를 다음과 같이 구분한다.

  - Presentation Layer: 유저 인터랙션과 관련된 부분 - User Interface라는 표현을 리네이밍한 것인데, 실제 유저 뿐만 아니라 GUI, CLI, API도 포함하는 의도로 변경되었다.

  - Application Layer: 도메인 객체에게 기능을 위임하는 부분

  - Domain Layer: 비즈니스 로직을 가지고 있는 부분
  - Infrastructure Layer: 메모리에 데이터를 영속화하고, 애플리케이션의 상태를 유지하는 부분

## Hexagon (Ports and Adpaters)
