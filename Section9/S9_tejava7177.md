# 빈 생명주기 콜백

스프링 빈의 동작과정은 `객체 생성 -> 의존관계 주입`  으로 동작함. 이후 데이터를 삽입 

데이터 삽입 순서가 잘못되면 정상적으로 데이터를 불러올 수 없음

⇒ 개발자는 의존관계 주입 단계가 끝난 시점을 파악하여 데이터를 삽입 해야 함.

## Callback

스프링은 의존관계 주입이 완료되면 **스프링 빈**에게 콜백 메서드를 통해서 초기화 시점을 알려줌.

스프링 컨테이너가 종료되기 직전에 소멸 콜백을 호출함.

**<스프링 빈의 이벤트 라이프 사이클>**

스프링 컨테이너 생성 → 스프링 빈 생성 → 의존관계 주입 → 초기화 콜백 → 사용 → 소멸전 콜백 → 스프링 종료 

- 초기화 콜백 : 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
- 소멸전 콜백 : 빈이 소멸되기 직전에 호출

# Spring Bean Callback 종류

### 기본코드

```java
public class BeanLifecycleTest {

    @Test
    public void lifecycletest() {
        //ConfigurableApplicationContext -> ac.close() 메서드를 사용하기 위해서.
        //ApplicationContext 는 제공하지 않음 / ConfigurableApplicationContext (상위 인터페이스)
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifecycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifecycleConfig {
        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }

}
```

## 인터페이스 InitializingBean, DisposableBean

```java
    //의존관계 주입이 끝나고 값 호출 -> 개발자가 손을 댈 수 없다는 단점
    @Override
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }

    @Override
    public void destroy() throws Exception {
        disconnect();
    }
```

### 특징

- 스프링 전용 인터페이스 → 코드가 스프링 인터페이스에 의존
- 초기화, 소멸 메서드의 이름을 변경할 수 없음
- 외부 라이브러리 적용 불가

### 결과

매우 고전적인 방법으로 콜백 기능을 사용하는 것이므로 현재로서는 전혀 사용하지 않음

## 빈 등록 초기화, 소멸 메서드 지정

```java
public class BeanLifecycleTest {

    @Configuration
    static class LifecycleConfig {
        @Bean(initMethod = "init", destroyMethod = "close")
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }

}
```

### 특징

- 메서드 이름을 자유롭게 변경 가능
- 스프링에 의존하지 않음
- 외부 라이브러리에도 적용 가능

<aside>
💡

@Bean 의 기본 값이 `Inferred` (추론) 으로 등록되어 있음.

만약 종료 메서드를 따로 정의하지 않으면 일반적으로 사용하는 `close` , `shutdown` 을 자동으로 호출해서 동작함

</aside>

## 애노테이션 @PostConstruct, @PreDestroy

```java
public class BeanLifecycleTest {

    @Configuration
    static class LifecycleConfig {   
        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }

}
```

```java
    @PostConstruct
    public void init(){
        System.out.println("NetworkClient.init");
        connect();
        call("init");
    }

    @PreDestroy
    public void close(){
        System.out.println("NetworkClient close");
        disconnect();
    }
```

### 특징

- 최신 스프링에서 권장하는 방법으로 매우 편리함
- 스프링에 의존적이지 않으며, 컴포넌트 스캔과 사용 가능
- 외부 라이브러리에서 사용 불가 → @Bean 으로 해결 가능