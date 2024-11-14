# Week 2-1. 객체지향스러운 아키텍처 설계하기
# Encapsulation; 캡슐화란?
- 객체의 속성과 행위를 하나로 묶고, 실제 구현 내용 일부를 외부로부터 감춰 은닉하는 기술
- **"public API를 구현하는데 사용되는 내부 상태와 행동을 내부로 숨긴다"**

![image](https://github.com/user-attachments/assets/014e2520-b4bc-4ce5-92cd-24cca693d92d)

<br>

### Access Modifier; 접근제어자
![image](https://github.com/user-attachments/assets/396f298d-0bae-4847-8e8a-c0c7000fd3ac)

<br>

## 왜 지켜야 할까?
변경의 유연함을 가지는 클래스 구조를 설계하기 위해

<br>

## 지키지 않았을 때 생기는 문제들
1. 유지보수의 어려움
2. 불변식이 깨져 예외 발생

<br>

## 결론
최대한 모든 요소를 private으로 선언하고, 접근 가능 범위를 한 단계씩(default > protected > public) 넓히는 방식으로 개발하는 것이 좋다.

<br>

# 추상화 vs 인터페이스
![image](https://github.com/user-attachments/assets/6c245b0f-85aa-48db-b582-7b7f1bff535a)

<br>


## 추상화 <<<<< 인터페이스
추상화보다 인터페이스가 좋다.
1. 상속으로 인한 결합도가 높아져, 변경의 전염성이 자식까지 전파
2. 불필요한 인터페이스까지 상속해, 자식 클래스의 내부 규칙을 무너뜨림
3. 클래스 조합 폭발
4. 메서드 오버라이딩의 오작용으로 의도와 다르게 동작해 오류 발생

<br>


## 그렇다면 인터페이스는 만사형통인가?
### Default 메서드
- 인터페이스에 새롭게 추가되는 메서드를 구현해 모든 구현체에게 기본적으로 제공하는 방식
- 기본 제공 기능을 변경해야 한다면, @Override하여 재구현 할 수 있다.
- 이때 모든 구현체가 문제 없이 사용할 수 있도록 설계 및 구현해야 한다.

<br>


### Objects 클래스
- 객체 비교, HashCode 생성, null 여부, 객체 문자열 리턴 등의 연산을 수행하는 정적 메서드로 구성
- `Objects.requireNonNull` : 메서드와 생성자에서 매개 변수 검증

<br>


# 변경하기 쉬운; SOLID
## DDD = OOP + SOLID
- Domain Driven Development의 약자로, 패턴/설계/아키텍처를 모두 합쳐 놓은 새로운 개발의 패러다임
- 가장 많이 사용하는 프로젝트 구조가 **데이터 중심**이라면, DDD는 **도메인 중심**이다.

<br>


## Domain; 도메인이란?
- SW로 문제를 해결하고자 하는 **비즈니스 영역**(이커머스; 핀테크; 금융 플랫폼; 성형 플랫폼)을 의미한다.
- 작은 의미로는 비즈니스를 구성하는 요소로, **유사한 업무의 집합체**(결제파트; 주문파트; 고객파트; 풀필먼트 파트)를 말한다.

<br>


## Event Stroming; 이벤트 스토밍
- 비즈니스와 관련된 모든 관계자가 한 자리에 모여 도메인 주요 이벤트를 중심으로 업무 프로세스를 정의해 비즈니스 프로세스와 정책, 경계, 도메인 등을 정의내리는 기법

<br>


## Aggregate와 Entity
- DDD에서 하나의 Aggregate(Entities + Value Objects)는 설계상 완전한 한 개의 도메인 영역(모델)을 표현
- 도메인 모델은 영속성을 가져야 함
- 우리가 정의한 Aggregate에 JPA와 관련된 어노테이션을 선언하고, ORM의 주체가 되어야 함
- 영속성 관리 Repository(jpa X)는 Aggregate 단위 별로 생성되어야 함 `데이터 일관성 유지`

<br>


## 고수준과 저수준 모듈
### 고수준 모듈
- 도메인에서 제공해야 하는 기능을 온전하게 제공하는 단일 기능(메서드)
- Application 계층의 Methods
예) paymentService#paymentApproved는 "PG사 API를 이용해 결제 승인을 요청한다." 라는 온전한 기능을 구현 및 제공한다.

<br>


### 저수준 모듈
- 작은 메서드로 구성되어 있으며, 독립적으로 온전한 기능을 제공할 수 없는 메서드
예) paymentService#paymentApproved의 로직을 실질적으로 수행하는 작은 메서드들

<br>


## SOLID의 DIP
의존 관계를 맺을 때 변하기 쉬운 것(대상)에 의존하기 보다, 쉽게 변하지 않는 것(인터페이스)에 의존하라는 원칙  

<br>

[변하지 않는 것, 변하는 것]
![image](https://github.com/user-attachments/assets/53ba753b-eb2f-4c7c-b497-40fe858d3463)

<br>

### ※ **변하기 쉬운 개념**에는 직접 의존하지 말자
![image](https://github.com/user-attachments/assets/e7d42a06-074b-4238-9b4b-b70bbfc8646f)

<br>

### SOLID의 DIP가 적용된 구조
![image](https://github.com/user-attachments/assets/ab63babc-9a30-40c9-8e83-6925f4e5ea7a)

<br>

## 헥사고날 아키텍처
전통적인 계층적 아키텍처의 단점을 보완해 **도메인 중심 구조**로 클린 아키텍처를 일반화

![image](https://github.com/user-attachments/assets/dd917b53-9035-4892-b693-46ce2b52af1f)

<br>

### 패키지 구조 - Representation; 표현
- 사용자의 요청 처리와 정보를 보여주는 역할
- 여기서 사용자는 웹 브라우저를 통해 Request를 보내는 Client, Front Server, OpenAPI 형태의 Request를 요청하는 시스템 등

<br>

### 패키지 구조 - Domain; 도메인
- 시스템(비즈니스)이 제공해야 하는 도메인 규칙
- 비즈니스의 핵심 로직 정의 및 구현
- 예) 구매 확정된 주문 건은 결제 취소가 불가능하다.

<br>

### 패키지 구조 - Application; 응용
- 사용자가 요청한 기능 실행
- 업무 로직을 직접 구현하지 않고, 도메인 계층을 조합해 기능 실행
- 도메인 계층에 구현된 비즈니스 핵심 규칙들을 조합하여 온전한 기능 제공

<br>

### 패키지 구조 - Infrastructure; 인프라
- Application의 Context 경계를 벗어나는 외부 시스템과의 연동 구현
- 인프라, 도메인: 저수준 모듈 구현 및 실제 기능 제공
- 표현, 응용: 고수준 모듈에 의존해 코드 작성

![image](https://github.com/user-attachments/assets/b99bf3f5-57b8-42bf-86cf-0c516ed89775)

<br>

# 변경하기 쉬운 구조
### 1. 멀티 모듈 구조
![image](https://github.com/user-attachments/assets/7c6a62b9-ae72-4881-88a7-10684727b0f8)

### 2. MSA 
![image](https://github.com/user-attachments/assets/43bf231a-0dd1-495d-a692-dd824cb61f86)
