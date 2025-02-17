# 빈 스코프

<aside>
💡

scope ?

: 빈이 존재할 수 있는 범위

</aside>

### Scope 의 종류

Singleton :  기본 스코프, 컨테이너의 시작과 종료까지 유지 되는 가장 넓은 범위의 스코프

Prototype : 프로토타입 빈의 생성과 의존관계 주입에만 관여하는 좁은 범위의 스코프

Web 관련 Scope

- request : 웹 요청이 끝날 때까지 유지되는 스코프
- session : 웹 세션이 끝날 때까지 유지되는 스코프
- application : 웹 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

# 프로토타입 스코프

![image.png](attachment:fc22d163-c001-4c92-ab7c-0f8cc9273249:image.png)

스프링 컨테이너에 요청한 시점부터 프로토타입 빈을 생성 후 의존관계를 주입

![image.png](attachment:c47613eb-ffc6-4d2c-b79c-7efad5b38725:image.png)

생성된 프로토타입 빈을 클라이언트에 반환 / 새로운 요청이 들어오면 반복

> 스프링 컨테이너는 **프로토타입 빈을 생성하고, 의존관계 주입, 초기화**를 처리한다. 
→ 프로토타입 빈을 관리하는 것은 클라이언트
+ 종료 메서드가 호출 되지 않는다. → 클라이언트가 직접 호출을 해야함
> 

## 싱글톤 빈과 함께 사용시 문제점

스프링에서 **Singleton 빈**과 **Prototype 빈**을 함께 사용하면, **싱글톤 빈의 생명주기와 프로토타입 빈의 특성이 충돌**하여 예상치 못한 동작이 발생할 수 있다.

 **< 싱글톤 빈의 특징 >**

• 스프링 컨테이너에서 **한 번만 생성되고 공유**됨.

• **생성 시점에만 의존 관계를 주입**받음.

• 같은 빈을 여러 곳에서 참조해도 동일한 인스턴스를 사용.

 **< 프로토타입 빈의 특징 >**

• 스프링 컨테이너에서 요청할 때마다 **새로운 인스턴스가 생성됨**.

• 스프링 컨테이너가 빈을 생성한 후 **생명주기 관리를 하지 않음**.

• 싱글톤 빈과 함께 사용될 경우, **싱글톤 빈이 생성될 때 프로토타입 빈을 주입받고 이후에는 새로 갱신되지 않음**.

<aside>
❓

 **정리**

**싱글톤 빈에서 프로토타입 빈을 의존성 주입 받으면 갱신되지 않는 문제**

• 싱글톤 빈은 **생성 시점에만** 의존 관계를 주입받기 때문에, 프로토타입 빈이 주입될 경우 **최초 주입된 프로토타입 빈만 계속 사용**된다.

• 이후 프로토타입 빈을 다시 사용하려 해도 **새로운 인스턴스를 생성하지 않고, 처음 주입된 인스턴스를 계속 사용**한다.

</aside>

## Provider 해결법

원하는 바 = 사용할 때 마다 항상 새로운 프로토타입 빈을 생성하기

```java
@Autowired
        private Provider<PrototypeBean> prototypeBeanProvider;
        

public int logic(){
            PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }

@Scope("prototype")
    static class PrototypeBean {
        private int count = 0;

        public void addCount(){
            count++;
        }

        public int getCount(){
            return count;
        }

        @PostConstruct
        public void init(){
            System.out.println("PrototypeBean.init" + this);
        }

        @PreDestroy
        public void destroy(){
            System.out.println("PrototypeBean.destroy" + this);
        }
    }
```

싱글톤  빈이 프로토타입을 사용할 때 스프링 컨테이너에 새로 요청하기 

**! 새로운 프로토타입 빈이 생성되기는 하나… 발생하는 문제점**

- 스프링 컨테이너에 종속적인 코드가 됨. → 단위 테스트가 복잡해짐.
- DL(Dependency Lookup) 의존관계 조회 기능보다 불필요한 부가적인 기능이 추가 됨.

### ObjectFactory , ObjectProvider

: 지정한 빈을 컨테이너 대신 찾아주는 기능 (DL 기능을 수행)

```java
public int logic(){
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject(); //provider를 적용함.
        prototypeBean.addCount();
        int count = prototypeBean.getCount();
        return count;
        }
```

→ .getObject() 는 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있음. (DL 기능)

스프링 기능을 사용하지만 비교적 단순한 방법으로 기능테스트나, mock 코드를 만들기 수월함.

**< 특징 >**

- ObjectFactory : 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
- ObjectProvider :  ObjectFactory 상속, 옵션, 스트림 처리 등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존
    
    = ObjectProvider 는 ObjectFactory 의 편의 기능이 발전된 것.
    

### JSR-330 Provider

: 자바 표준의 JSR-330 이므로 gradle 에 의존성을 주입해야함.

```java
public int logic(){
	            PrototypeBean prototypeBean = prototypeBeanProvider.get(); //JSR-330
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
```

**< 특징 >**

- provider.get() 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있음. (DL)
- 자바 표준이기 때문에 기능이 단순, mock 코드로 구현하기 쉬움.
- 필요한 기능만을 제공
- 자바 표준이기 때문에 다른 컨테이너에서도 사용가능하다는 장점, 별도의 라이브러리가 필요하다는 단점

# 웹 스코프

: 웹 환경에서만 동작하는 웹 스코프, 스프링이 해당 스코프의 종료 시점까지 관리 → 종료 메서드가 호출 됨. 

# request 스코프 예제

: HTTP 요청 하나가 끝날 때까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리

![image.png](attachment:a7d59274-18a3-4514-8928-b795194e849b:image.png)

<aside>
❓

웹 서비스를 운영할 때 동시에 HTTP 요청이 오면 정확히 어떤 요청이 남긴 로그인지 알기 힘들다.

→ request 스코프를 사용하여 UUID {message} 형식으로 구분하고 싶음.

</aside>

```java
@Component
@Scope(value = "request" , proxyMode = ScopedProxyMode.TARGET_CLASS)       //속성이 두 개 이상인 경우 value 사용
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestURL + "] " +  message);
    }

    @PostConstruct
    public void init() {
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create : " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "]" + "[close]" + this);
    }
}
```

<aside>
💡

**< 정리 >**

@Scope(value = “request”) 를 사용하여 스코프 지정. → 요청 당 1개의 빈이 생성됨

@PostConstruct 초기화 메서드를 사용하여 uuid 생성 및 저장

@PreDestroy 사용 → 종료 메서드 호출

</aside>

### < 결과 (오류 발생) >

![image.png](attachment:be3b3eec-5301-4d86-9677-0d5b494ea48f:image.png)

: 스프링 애플리케이션을 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, request 스코프 빈은 아직 생성되지 않는다. → 고객의 요청이 와야 생성할 수 있음.

# 스코프와 Provider

ObjectProvider를 사용하여 문제 해결

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final ObjectProvider<MyLogger> myLoggerProvider;
    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }

//        private final MyLogger myLogger;
//
//        public void logic(String id){
//            myLogger.log("service id = " + id);
//        }

}

```

**< 결과 >**

![image.png](attachment:27e6fbc8-f26a-412d-8937-bc6f995e2f3c:image.png)

<aside>
💡

**설명**

- ObjectProvider 사용으로 request scope 의 빈 생성을 지연할 수 있음.
- .getObject() 호출 시점에는 HTTP 요청이 진행중이므로 requset scope 빈의 생성은 정상처리
</aside>