# Spring Container

하나의 Interface 개념이며, XML 기반으로 만들 수도 있고, 애노테이션 기반의 자바 설정 클래스 (AppConfig) 로 만들 수 있다.

<aside>
💡

Spring Container 는 `BeanFactory`, `ApplicationContext` 로 구분 할 수 있다.

</aside>

## Spring Container 생성과정

1. 스프링 컨테이너 생성
2. 스프링 빈 등록
3. 

### Spring Bean 데이터 조회

1. 모든 Bean 출력하기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/eb36e72a-a8d5-41d8-b80e-b90d54a3b883/image.png)

1. 애플리케이션 빈 출력하기 (개발자가 정의한)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/0b9e6b0a-b530-4a95-a043-f5e81bf2142a/image.png)

<aside>
💡

ac.getBeanDefinitionNames () : 스프링에 등록된 Bean 정보 조회

ac.getBean() :  빈 이름으로 된 빈 객체 조회

</aside>

### Spring Bean 조회 - 기본

**조회 방법**

- ac.getBean ( Bean Name, Type)
- ac.getBean( Type )

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/0015bec9-b755-40a4-a8eb-6694306d1c57/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/cc1f44e6-fc7c-499e-8f59-c11a378c1406/image.png)

> 만약 같은 타입의 Bean 을 호출할 경우?
오류가 발생한다, → 이는 Bean 이름 지정으로 해결할 수 있다.
> 
> 
> ![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/067e4d29-1e55-4bfc-8a99-f09c761a78d7/image.png)
> 

## BeanFactory & Application Context

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/44de3c12-c9c3-4f1b-b8d0-b433ad742ad8/image.png)

**BeanFactory**

Spring Container의 최상위 인터페이스

Spring Bean을 관리하고 조회하는 기능

getBean 제공

**ApplicationContext**

- BeanFactory 기능을 모두 상속 받아 제공
- Bean을 관리하고 검색함

<aside>
💡

둘의 차이점?

ApplicationContext 는 부가기능을 제공한다.
메시지 소스를 활용한 국제화 기능

- 메시지 소스를 활용한 국제화 기능
- 환경 변수 (로컬, 개발, 운영 등) 구분해서 처리
- 애플리케이션 이벤트 : 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- 편리한 리소스 조회 : 파일 클래스패스, 외부 등에서 리소스를 편리하게 조회

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/088c0c16-c6ff-43f8-a152-ad6f71612bcf/image.png)

</aside>

## Spring Bean 설정 메타 정보

BeanDefinition(설정 메타정보) : 스프링에서 다양한 설정 형식을 제공하는 추상화

++ 역할과 구현을 개념적으로 나눈 것. Bean 개념을 추상화

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/e3d1505b-3a20-42d4-b075-2ae0df3d81f4/image.png)

BeanDefinition 은 자바 코드 , XML 파일, 등으로 이루어질 수 있다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/9746a81a-7b06-4dbd-9700-e58abf4b4ff3/image.png)

**BeanDefintion 정보**

- BeanClassName  : 생성할 빈의 클래스 명
- factoryBeanName : 팩토리 역할의 빈을 사용할 경우 이름
- factoryMethodName : 빈을 생성할 팩토리 메서드 지정
- Scope : 싱글톤 (기본 값)
- lazyInit : 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연 처리하는지 여부
- InitmethodName : 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드
- DestoryMethodName : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드
- Constructor arguments, Properties : 의존관계 주입에서 사용

> Spring 은 BeanDefinition 이라는 것으로 Spring Bean의 설정 메타 정보를 추상화한다.
>