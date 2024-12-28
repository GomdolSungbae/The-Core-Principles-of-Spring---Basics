
## 스프링 컨테이너 생성

스프링 컨테이너는 `ApplicationContext` 인터페이스를 구현한 다양한 컨테이너 클래스 중 하나를 이용하여 생성된다. 

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

1. 스프링 컨테이너 생성
	- AppConfig.class를 기반으로 스프링 컨테이너가 생성됨
	- 설정 정보(`@configuration` 및 `@Bean`)가 포함된 자바 클래스를 로딩
2. 구성 정보 활용
	- 설정 클래스인 AppConfig.class에서 정의된 빈(Bean) 정보를 스캔하고 등록
	- 스프링 빈 저장소에 빈 이름과 객체가 저장
		- 해당 저장소는 스프링의 BeanFactory 또는 ApplicationContext가 관리

#### 빈 저장소 (Bean Repository)
```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
UserService userService = context.getBean("userService", UserService.class);
```
- 스프링 컨테이너는 빈 객체를 이름(키)와 함께 저장
- 사용자가 필요할 때 빈을 의존성 주입(DI)또는 getBean() 메서드로 불러올 수 있음

#### 스프링 빈 등록 
```java
@Bean
public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
}

@Bean
public OrderService orderService() {
    return new OrderServiceImpl(memberRepository(), discountPolicy());
}

@Bean
public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
}

@Bean
public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
}

```
- memberService: `MemberServiceImpl` 객체 생성 및 등록
- orderService: `OrderServiceImpl` 객체 생성 및 등록
- memberRepository: `MemoryMemberRepository` 객체 생성 및 등록
- discountPolicy: `RateDiscountPolicy` 객체 생성 및 등록

#### 스프링 빈 의존관계 설정 

```java
@Bean
public MemberService memberService() {
    return new MemberServiceImpl(memberRepository());
}

@Bean
public OrderService orderService() {
    return new OrderServiceImpl(
        memberRepository(),
        discountPolicy()
    );
}

@Bean
public MemberRepository memberRepository() {
    return new MemoryMemberRepository();
}

@Bean
public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
}

```

- `@Bean` 어노테이션
    - 스프링 컨테이너에 빈(Bean)을 등록
    - 메서드 이름이 빈의 이름이 되며, 반환 객체가 빈 객체가 됨
- 의존성 주입 구조
    - `memberService()` → `memberRepository()` 의존성 주입 
    - `orderService()` → `memberRepository()` 및 `discountPolicy()` 의존성 주입 


## 컨테이너에 등록된 모든 빈 조회 

#### 스프링 컨테이너에 등록된 모든 빈을 출력하는 테스트 코드 
```java
class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력하기")
    void findAllBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object = " + bean);
        }
    }
}

```
- `AnnotationConfigApplicationContext` : 자바 기반 설정 클래스를 사용하여 스프링 컨테이너를 생성
- `AppConfig.class` : 빈 설정 정보를 제공하는 클래스
	- `@Configuration`과 `@Bean` 어노테이션을 통해 빈을 정의





#### 스프링 컨테이너에 등록된 애플리케이션 빈을 출력하는 테스트 코드
```java
@Test
@DisplayName("애플리케이션 빈 출력하기")
void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

        if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object = " + bean);
        }
    }
}

```

- `BeanDefinition`: 빈의 메타데이터(정의 정보)를 저장하는 객체
- `getRole()`: 빈의 역할을 구분
	- `ROLE_APPLICATION`: 개발자가 직접 등록한 빈 또는 외부 라이브러리에서 등록한 빈
	- `ROLE_INFRASTRUCTURE`: 스프링 내부에서 관리하는 지원 빈(자동 설정, 처리기 등)
- `ROLE_SUPPORT`: 인프라 빈(스프링 컨테이너 지원을 위한 내부 빈)


## 스프링 빈 조회 

#### 특정 빈 이름으로 조회하기 
```java
class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName() {
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        System.out.println("memberService = " + memberService);
        System.out.println("memberService.getClass() = " + memberService.getClass());
    }
}
```

- `getBean()`
	- 첫 번째 인자: 빈 이름(`memberService`)
	- 두 번째 인자: 빈 타입(`MemberService.class`)
	- 지정한 이름과 타입을 가진 빈을 스프링 컨테이너에서 조회

#### 타입만으로 빈 조회하기 
```java
@Test
@DisplayName("이름 없이 타입으로만 조회")
void findBeanByType() {
    MemberService memberService = ac.getBean(MemberService.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
}
```
- **`getBean(Class<T> requiredType)`**
	- 빈 이름 없이 **타입만으로 빈을 조회**
	- 해당 타입의 빈이 하나만 등록되어 있어야 함
	- 같은 타입의 빈이 두 개 이상 등록되어 있으면 `NoUniqueBeanDefinitionException` 발생

#### 구체 타입으로 빈 조회하기
```java
@Test
@DisplayName("구체 타입으로 조회")
void findBeanByName2() {
    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
}
```
- 인터페이스가 아닌 구현 클래스 타입으로 빈을 조회
- **`getBean(String name, Class<T> requiredType)`**
	- 첫 번째 인자: **빈 이름** (`memberService`)
	- 두 번째 인자: **구체 타입** (`MemberServiceImpl.class`)
	- 지정된 이름과 타입에 해당하는 빈을 스프링 컨테이너에서 조회


## BeanFactory와 ApplicationContext


## 다양한 설정 형식 지원 


## 스프링 빈 설정 메타 정보

