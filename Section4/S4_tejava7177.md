<aside>
💡

### 애자일 소프트웨어 개발 선언

: 계획에 따른다기보단 변화에 대응하기를 가치있게 여긴다.

</aside>

# Test 코드

`test 파일 자동 생성` : cmd + shift + t 

**++ Test 는 실패 Test 도 진행해야한다.**

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/ac8a5ac7-c30d-4da4-907e-797383ac9afd/image.png)

Assertions `option + Enter` ⇒ Add on demand = 코드가 간결해지고 효율적임

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/391fdfc7-41bf-427a-a06d-73d03a009c5c/image.png)

`@BeforeEach` : 각 test 실행전에 무조건 먼저 실행하게 함.

# OCP / DIP 를 위반하지 않고 기능을 확장하는 방법

추상에만 의존하도록 변경

DIP 가 위반하지 않도록 Interface 에만 의존하도록 함 (설계변경) ⇒ `관심사의 분리` 

# 관심사의 분리

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e9d32869-f809-4b19-b32a-c3372ad0c161/aab94461-1f28-48c6-a890-9f82daec34c1/image.png)

<aside>
💡

## AppConfig

: 애플리케이션의 전체 동작 방식을 구성 config 하기 위해 “구현객체를 생성”하고 “연결” 하는 책임을 가지는 별도의 설정 클래스

⇒ 애플리케이션의 전반적인 운영을 책임진다. 전체 설정 & 구성

**기존 Service Code에 생성자를 생성 → 생성자를 통해서 구현 객체를 결정**

</aside>

## Result = DIP 만족 (생성자 주입)

final은 필수적으로 필드 값이 할당되어야 함. (+ 생성자 포함) 

cmd + n : `생성자 생성`

cmd + e : 과거 history 탐색 (이전 코드 탐색)

cmd + option + v : 자동완성

### AppConfig

애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다. + 생성한 객체 인스턴스의 참조 (레퍼런스)를 생성자를 통해서 주입(연결) 해준다 

= 생성자 주입 DI (`Dependency Injection`) 의존관계(의존성) 주입

### AppConfig 과정 정리

AppConfig 에서 객체를 생성하고 ‘참조값’과 함께 생성자에게 전달 → 이후 test 코드로 검토 및 수정

### AppConfig 적용

> AppConfig 의 등장으로 요구사항이 변경되면 `사용영역`과 `구성영역`으로 나눠진다.
사용영역 → 수정할 필요가 없음
구성영역 → 이 부분만 수정하면 됨.
> 

# IoC , DI Container

## 제어의 역전 Inuersion of Control

: Framework 가 개발자 대신 호출해 주는 것?

⇒ 구현 객체는 자신의 조직을 ‘실행’하는 역할만 가지고 있다. ex) AppConfig

## Framework

: 개발자가 작성한 코드를 제어하고, 대신 실행해줌

ex) JUnit

## Library

: 개발자가 작성한 코드가 직접 제어 흐름을 담당하는 것

## 의존관계 주입 DI Dependency Injection

<aside>
💡

## 의존관계

: `정적인 클래스 의존관계` 와 실행시점에 결정되는 동적인 객체 (인스턴스)간의 의존관계를 분리함.

</aside>

> **정적인 의존관계**
(import) Code 만 보고 판단할 수 있음.
> 

> **동적인 객체 인스턴스 의존관계**
애플리케이션 실행시점에 생성 된 객체 인스턴스의 참조가 연결 된 의존관계
>