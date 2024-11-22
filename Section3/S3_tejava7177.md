![startSpring.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/d5ce0045-2869-4d31-9277-87dd8d10c771/startSpring.png)

[start.spring.io](http://start.spring.io) 해당 프로젝트의 버전 및 이름 설정

![gradle설정이유.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/84e994c8-085a-452b-a1eb-ca30a6ef1478/gradle%E1%84%89%E1%85%A5%E1%86%AF%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%B2.png)

<aside>
💡

Setting 의 `Build and run using` `Run tests using`  을 `IntelliJ IDEA` 로 설정하면 IntelliJ 에서 자체적으로 프로젝트를 실행하기 때문에 동작 속도가 빠르다.

</aside>

# 비지니스 요구사항 정리

## 목표

**역할 (Interface) 와 구현(구현 객체)을 나누는 것**

### 회원 도메인 설계

![IMG_2619.heic](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/a8a84ba3-9ca6-4874-b053-eeda2adb6a1b/IMG_2619.heic)

회원 저장소 (역할, Interface) 를 만듦. → 일단 메모리 회원저장소(비교적 간단한 구현 방식) 구현

why?

`자체 DB 구현 방식` 으로 할 것인지 `외부 시스템 연동 회원 저장소` 로 할 것인지 구현체가 정해지지 않음

⇒ 이후 구현 (구현 객체 ) 가 정해지면 해당 구현 방식을 적용하는 방식

### 회원 Class Diagram

++ 회원 클래스 다이어그램은 동적인 변경요소가 많아 프로젝트를 정확히 표현하기 어려움.

![IMG_2620.heic](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/671bda13-da8a-4c59-8c2f-1adfb9bf25f1/IMG_2620.heic)

### 회원 객체 다이어그램

: 객체 간의 메모리 참조의 관계를 보여주는 다이어그램 (서버가 실제 사용하는 Instance 를 나타냄.)

![IMG_2621.heic](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/1ad0c193-26ca-47eb-b53e-8ae7a275591c/IMG_2621.heic)

구현체가 하나만 있을 경우 ~Impl 로 명시함.

![스크린샷 2024-11-22 오후 12.25.49.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/8e08b689-83b5-47a0-8f8b-443fbacd4b27/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-22_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12.25.49.png)

<aside>
💡

현재 MemberServiceImpl 는 Ocp 와 DIP 를 만족하고 있는가?

⇒ 아니다. 현재 상태는 구현체와 추상화 모두에 의존하고 있다

</aside>

![스크린샷 2024-11-22 오후 12.27.37.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/c4ec611b-1979-4ffa-b931-d286dca3f9f4/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-22_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12.27.37.png)

<aside>
💡

OrderServiceImpl 는 `단일체계 원칙`을 잘 고수함 

⇒ 할인정책이 변경(ex. 정액할인 → 정률 할인 )되더라도 주문 서비스를 고칠 필요가 없다.

</aside>

---

## HashMap<>() 에 대한 동시성 이슈?

![스크린샷 2024-11-22 오후 12.11.49.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/f7b22aea-f404-4e51-be0d-a13dbfadcb6a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-22_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12.11.49.png)

- **`Map<Long, Member>`**:
    - 데이터를 **`key-value`** 형태로 저장하는 자료구조
    - 여기서 `Long`은 `key`, `Member`는 `value`를 나타낸다.
- **`HashMap<>`**:
    - `HashMap`은 Java에서 제공하는 기본 Map 구현체로, `key`와 `value`를 해시 기반으로 저장
    - `HashMap`은 비동기적(Non-synchronized)이다. 즉, **다수의 스레드가 동시에 접근하면 데이터가 손상될 가능성**이 있다.
- **`static` 키워드**:
    - 클래스 레벨에서 `store` 변수를 공유하기 때문에, 어떤 객체에서도 동일한 `store` 맵을 접근할 수 있다.
    - 따라서 여러 스레드가 동시에 `store`에 접근할 가능성이 높아 동시성 문제가 발생할 수 있다.

### 동시성 이슈란?

<aside>
💡

- **동시성 문제**:
    - `HashMap`은 **스레드 안전하지 않습니다**. 즉, 여러 스레드가 동시에 `store.put()`이나 `store.get()`을 호출하면 데이터 손실, 덮어쓰기, 또는 예외가 발생할 수 있습니다.

### 예시:

```java

Thread 1: store.put(1L, new Member("A"));
Thread 2: store.put(1L, new Member("B"));
```

- 두 개의 스레드가 동시에 같은 `key`에 값을 넣으려고 하면, **하나의 값이 덮어써지거나, 중간 상태가 발생**할 수 있습니다.

</aside>

### 해결 방법: `ConcurrentHashMap` 사용

- **`ConcurrentHashMap`**:
    - `ConcurrentHashMap`은 `HashMap`과 동일한 기능을 제공하지만, **스레드 안전하다.**
    - 내부적으로 분리된 **버킷(bucket)** 구조를 사용하여, 여러 스레드가 동시에 접근해도 안전하게 동작

```java
private static Map<Long, Member> store = new ConcurrentHashMap<>();
```

---

### 4. `ConcurrentHashMap`을 사용하면 어떻게 동시성 문제가 해결되는가?

- `ConcurrentHashMap`은 **세부적인 동기화**를 제공
    - 특정 **버킷**만 잠금(lock)을 걸어 다른 스레드의 접근을 차단
    - 동시에 다른 버킷에는 다른 스레드가 접근 가능하므로 성능도 유지

### 동작 방식:

- **쓰기 작업**(`put`):
    - 동일한 `key`에 쓰는 작업은 원자적으로 처리 즉, 중간 상태 없이 완전히 완료
- **읽기 작업**(`get`):
    - 읽기 작업은 동기화가 필요하지 않아 빠르게 동작

# Test

## JUnit = TestFramework

![스크린샷 2024-11-22 오후 12.21.56.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/f401cee3-dde6-413f-99da-8da0241f5be7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-11-22_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12.21.56.png)

Given - When - Then 으로 나누어 `단위 테스트`를 진행한다.

⇒ Spring 이나 Container 도움없이 순수 Java 코드로 테스트를 실행하는 것

# 🍯 소소한 Tip

## 생성자, Getter & Setter (데이터 가져오기 옮기기)

`cmd + n` + `cmd + a`

`psvm` : public static void main ~

`soutv` : System.out.println~~

`Fn + F2` : 오류난 곳으로 이동 

`cmd + option + v` : 코드 자동 완성