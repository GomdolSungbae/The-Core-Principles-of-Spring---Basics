# Section 2 - 객체 지향 설계와 스프링

# Spring Framework

**핵심 기술** : 스프링 DI 컨테이너 , AOP, 이벤트

**웹 기술** : 스프링 MVC, WebFlux

**데이터 접근 기술** : 트랜잭션, JDBC, ORM 지원

**기술 통합** : 캐시, 이메일, 원격접근

**테스트** : 스프링 기반 테스트 지원

**언어** : 코틀린, 그루비

# SpringBoot

Spring 의 여러 기술들을 손쉽게 사용할 수 있음.

최근에는 기본으로 사용

- Tomcat 웹서버를 내장해서 자체적으로 실행가능
- Starter 종속성 제공
- 외부라이브러리(ThirdParty) 의 버전을 신경쓰지 않아도 됨
- 메트릭, 모니터링 기능 제공

# Spring 을 일컫는 말?

- Spring DI Container 기술
- Spring Framework , Spring Boot 전체 생태계

# Spring 의 핵심개념

객체지향의 언어의 특징을 살리는 framework

좋은 객체지향 애플리케이션을 개발할 수 있게 도와주는 framework

## 좋은 객체 지향 애플리케이션 feat.  객체지향프로그래밍의 특징

- 추상화
- 캡슐화
- 상속
- 다형성 Polymorphism

<aside>
💡

## 다형성 Polymorphism

Client 에게 영향을 주지 않고 새로운 기능을 제공할 수 있다.

Why?

역할과 기능으로 구별했기 때문 → 대체 가능 = 유연하고 변경에 용이

++ `Interface(역할)` 를 안정적이게 설계하는 것이 중요하다. ++

</aside>

> **자바의 다형성
: Overriding → 결국 Overriding 된 메서드가 실행**
> 

---

# 좋은 객체 지향 설계 5가지 원칙 (SOLID)

- SRP (Single Responsibility Principle)
- OCP (Open / Closed Principle)
- LSP (Liskov Substitution Principle)
- ISP (Interface Segregation Principle)
- DIP (Dependency Inversion Principle)

## SRP 단일 책임 원칙

하나의 클래스는 하나의 원칙을 가져야한다. ⇒ 변경이 있을 때 파급 효과가 적은 것

## `OCP 개방 폐쇄 원칙`

다형성을 활용하는 것

인터페이스를 구현한 새로운 클래스를 하나 만들어서 새로운 기능 구현

## LSP 리스코프 치환 원칙

단순한 컴파일 성공을 넘어 Interface 간의 구약을 지키는 것

## ISP 인터페이스 분리 원칙

특정 Client 를 위한 interface 여러개가 범용 interface 1개 보다 낫다.

## `DIP 의존관계 역전 원칙`

Client Code 가 구현체가 아닌 interface 를 바라보게 함

## 다형성만으로는 `OCP` , `DIP` 를 지킬 수 없다.

Spring의 DI (Dependency Injection) / DI Container 를 통해 해결 할 수 있다.

---

> 실무적 고민
> 

If 기능을 확장할 가능성이 없다면?

⇒ 구체 클래스를 직접 사용하지 않고, 향후 꼭 필요할 때 리펙토링해서 인터페이스를 도입한다.