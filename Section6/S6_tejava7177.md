# 웹 애플리케이션과 싱글톤

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/a43cca4a-e48d-486f-bc56-b70d26305dcf/image.png)

웹 애플리케이션 서비스는 각 다른 클라이언트가 동시에 여러 요청을 보내는 경우가 대부분
매번 다른 객체를 생성하는 방법은 매우 비효율적임 ⇒ 스프링의 DI 컨테이너를 통해 객체를 1개만 생성하고 공유하여 효율성을 높임

# 싱글톤 패턴

: 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴

<aside>
💡

private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하여 객체를 생성하지 못하도록 한다.

```java

public class SingletonService {

    // 자기 자신을 내부에 private static으로 가지고 있음
    // 1. static 영역에 객체 instance를 미리 하나 생성해서 올려둔다.
    private static final SingletonService instance = new SingletonService();

    // 2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용
    public static SingletonService getInstance() {
        return instance;
    }

    // 3. 생성자를 private로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다.
    private SingletonService() {
    }

    public void logic() {
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```

</aside>

# 싱글톤 방식의 주의점

- 싱글톤 패턴을 구현하기 위한 추가적인 코드가 필요
- 의존관계 상 클라이언트가 구체 클래스에 의존 → DIP 위반
- 클라이언가 구체 클래스에 의존해서 OCP 원칙을 위반 할 가능성이 있음
- 테스트하기 어려움
- 내부 속성을 변경하거나 초기화 하기 어려움
- 유연성이 떨어짐

## 싱글톤 컨테이너

싱글톤 + 스프링 컨테이너 = 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리함.

스프링 컨테이너는 싱글턴 패턴의 모든 단점을 해결 및 DIP, OCP 등에도 위반하지 않음

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/add4a72b-7bcc-43d8-ad8f-7be8b983b1da/image.png)

<aside>
💡

싱글톤 패턴을 구현하는 방법

- 객체를 미리 생성 → 가장 안전하고 단순함
- 인스턴스 조회 호출 시 객체 생성
- 지연처리 : 있으면 쓰고, 없으면 생성하는 방식
</aside>

## 싱글톤 방식의 주의점

객체 인스턴스 하나만 생성하여 여러 클라이언트가 공유하기 때문에 상태 유지(stateful)가 아닌 **무상태(stateless)**로 설계해야함

- 특정 클라이언트에 의존적인 필드가 있으면 안 됨.
- 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안 됨.
- 가급적 읽기만 가능해야 함. → 수정 불가
- 필드 대신 자바에서 공유되지 않은 **지역 변수, 파라미터, ThreadLocal** 등을 사용

<aside>
💡

잘못 된 예시

```java
public class StatefulService {

    private int price; 
    
    public void order(String name, int price) {
        System.out.println("name = " + name + "price = " + price);
        this.price = price; 
    }
    
    public int getPrice() {
        return price;
    }
}
```

해결 방안

```java
public class StatefulService {

    //private int price;

    public int order(String name, int price){
        System.out.println("name = " + name + "price = " + price);
        //this.price = price;
        return price;
    }

//    public int getPrice() {
//        return price;
//    }

}
```

</aside>

---

# @Configuration

@Configuration 이 붙은 클래스에서 @Bean 이 붙은 메서드 객체는 스프링의 컨테이너에서 싱글톤으로 관리 된다.

```java
@Configuration
public class AppConfig {

    @Bean  //스프링 컨테이너 등록
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy(); //정액 할인
    }

}
```

AppConfig 코드를 보면 MemoryMemberRepository 가 생성되면서 싱글톤에 위배되는 것처럼 보임.

하지만 로그를 확인해보면 같은 인스턴스를 참조하며 실제로는 1번만 호출

그 이유는…

@Configuration 사용과 바이트 코드에  있다.

```java
//configurationDeep()
@Test
    void configurationDeep() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean = " + bean.getClass());
    }
```

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/161c6149-2818-4715-ad1d-54c75749c353/image.png)

@CGLIB 는 스프링이 바이트코드 조작 라이브러리를 사용 

: 개발자가 만든 클래스가 아니라 스프링이 CGLIB 라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고 스프링 빈으로 등록

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/7f683efa-2766-4daf-b69f-a44c5eb66dc8/image.png)

 

---

## @Configuration 의 유무차이

스프링 빈은 정상적으로 등록이 가능하지만, 싱글톤을 보장하지 않음 → 객체를 매번 호출하고 생성하며 주입 됨. ⇒ 결국은 @Configuration 을 잘 활용하자