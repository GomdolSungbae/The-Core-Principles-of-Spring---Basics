# 컴포넌트 스캔과 의존관계 자동 주입

## Component Scan?

스프링 빈의 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 기능

++ @Autowired : 의존관계 자동 주입 (다음 Section 에서 자세히 설명) ++

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import static org.springframework.context.annotation.ComponentScan.
*;
@Configuration
@ComponentScan(
excludeFilters = @Filter(type = FilterType.ANNOTATION, classes =
Configuration.class))
public class AutoAppConfig {

}
```

설정 정보에 @ComponentScan 추가 후 프로젝트의 Repository, 구현체, Service 코드(@Autowired 도 추가)에 @Component 설정정보 추가

# 탐색 위치와 기본 스캔 대상

## 탐색할 패키지의 시작 위치 지정

```java
@Configuration
@ComponentScan(             //자동으로 spring bean 주입 @Component어노테이션을 찾아서..

        //시작 위치 설정 -> 하지만 권장하지 않음 프로젝트 최상단 경로로 설정하는 것이 권장방법
        basePackages = "hello.core.member",
        basePackageClasses = AutoAppConfig.class,

        //AppConfig 코드(수동으로 Spring Bean 주입)에 @Configuration 이 붙어 있음 -> 충돌 방지를 위해 제외하는 필터
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
)
```

<aside>
💡

**탐색할 패키지의 시작 위치의 관례**

프로젝트를 시작하기 위해 가장 최상위 코드인 @SpringBootApplication 을 시작 위치에 두는 것이 관례인데 해당 어노테이션에 @ComponentScan 이 포함되어 있음 → 시작위치로 지정하는 것을 권장

</aside>

## Test

```java
@Test
    void basicScan(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
        MemberService memberService = ac.getBean(MemberService.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
    }
```

## 동작과정

### 컴포넌트 스캔

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/5eccd9d1-cb2c-4597-bd8e-86c760e30151/image.png)

@ComponentScan 이 @Component 가 붙은 모든 클래스를 스프링  빈으로 등록

기본 클래스 명을 그대로 사용하되 앞 글자만 소문자로 변경

### 의존관계 자동주입 (@Autowired)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/272d0a8a-e728-45c6-9533-c6dee866b7b5/image.png)

생성자에 @Autowired 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입

과정은 getBean(MemberRepository.class) 와 얼추 비슷함.

## 컴포넌트 스캔 기본 대상 및 추가 기능

- @Component : 컴포넌트 스캔에서 사용
- @Controller : 스프링 MVC 컨트롤러에서 사용 + 스프링 MVC 컨트롤러 인식
- @Service : 스프링 비지니스 로직에서 사용 + 핵심 비지니스 로직으로 인식할 수 있음
- @Repository : 스프링 데이터 접근 계층에서 사용 + 스프링 데이터의 저근 계층으로 인식, 데이터 계층의 예외를 스프링 예외로 변환
- @Configuration : 스프링 설정 정보에서 사용 + 스프링 빈이 싱글톤을 유지하도록 추가 처리

<aside>
💡

Annotation 은 상속 관계나 , 서로 연동되는 관계가 아님. 

→ 자바 언어로 연결되어 있는 것처럼 보이지만, 자바 언어의 특성이 아닌 Spring 에서 유연하게 상호작용할 수 있도록 지원하는 기능

</aside>

# 필터

includeFilters : 컴포넌트 스캔 대상을 추가로 지정

excludeFilters : 컴포넌트 스캔에서 제외할 대상 지정

### 어노테이션 생성

```java
import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
//제외할 필터
public @interface myExcludeComponent {
}
```

### 클래스 생성

```java
@myExcludeComponent
public class BeanB {
}
```

### 테스트 코드 생성

```java
public class ComponentFilterAppConfigTest {
    @Test
    void filter() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
        BeanA beanA = ac.getBean("beanA", BeanA.class);

        assertThat(beanA).isNotNull();

        Assertions.assertThrows(
                NoSuchBeanDefinitionException.class,
                () -> ac.getBean("beanB", BeanB.class));

    }

    @Configuration
    @ComponentScan(
            includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = myincludecomponet .class),
            excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = myExcludeComponent.class)
    )
    static class ComponentFilterAppConfig {

    }
}
```

## FilterType

- ANNOTATION : 기본값, 애노테이션 인식해서 동작
- ASSIGNABLE_TYPE : 지정한 타입과 자식 타입을 인식해서 동작
- ASPECTJ : AspectJ 패턴 사용
- REGEX : 정규 표현식
- CUSTOM : TypeFilter 이라는 인터페이스 구현해서 처리

# 중복 등록과 충돌

### 자동 빈 등록 vs 자동 빈 등록

: 비교적 간단한 오류 발생 `ConflictingBeanDefinitionException` 를 확인하여 쉽게 수정할 수 있음.

### 수동 빈 등록 vs 수동 빈 등록

: **수동으로 등록 한 빈**에 우선권을 가짐. → 자동 오버라이딩이 발생함.

```java
public class AutoAppConfig {

    //수동 vs 자동 의존관계 주입 -> overriding 발생
    @Bean(name = "memoryMemberRepository")
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

}
```

만약 AutoAppConfig 를 실행한다면 Test 코드는 정상적으로 실행함.

하지만 Boot 로 실행하면 다음과 같은 오류 발생

### 오버라이딩 오류 발생

> The bean 'memoryMemberRepository', defined in class path resource [hello/core/AutoAppConfig.class], could not be registered. A bean with that name has already been defined in file [/Users/simjuheun/Desktop/개인프로젝트/Spring_Core/core/out/production/classes/hello/core/member/MemoryMemberRepository.class] and overriding is disabled.
> 

<aside>
💡

만약 해결하고 싶다면? propertiese 코드에서 

**spring.main.allow-bean-definition-overriding=true** 코드를 추가

⇒ 오버라이딩은 발생하지만 오류를 발생시키지는 않음. (권장 x)

</aside>