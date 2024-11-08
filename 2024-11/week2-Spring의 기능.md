# Week 1-2. Spring의 기능 자세히 살펴보기
#### 왜 Spring Boot가 등장했을까?
톰캣을 여러 개 설치하면 모두 환경이 달랐다 => tomcat을 내장시켜서 동일한 환경을 만들자 

# Request 처리 프로세스
![image](https://github.com/user-attachments/assets/2d29df46-2064-4431-a625-60b4fd2ad6a3)

- **인증**은 `Filter`에서 디스패처 서블릿이 요청을 받기 전에 사용자를 확인한다.
- **인가**는 인증된 사용자가 컨트롤러에 접근 권한이 있는지 `Intercepter`에서 확인한다. 

1. 프론트 컨트롤러인 `Dispathcher-Servlet`이 가장 먼저 요청 메세지를 받는다.
2. `Handler Mapping` : Requset Message의 Header에 저장된 부가 정보를 이용해 요청 처리를 위임할 Controller를 찾는다.
3. Handler Adapter에게 찾은 Controller를 전달한다.
    > **Handler Mapping이 직접 요청을 위임하지 않는 이유는,**  
    > - Controller의 구현 방식이 다양해 HandlerAdapter라는 Adapter 인터페이스를 통해 어댑터 패턴을 적용
    > - 구현 방식에 상관 없이 요청을 위임할 수 있는 것
4. `HandlerAdapter` : Controller에 요청을 위임한다.
    - Controller의 Method 인자에 선언된 객체 (@RequestBody, @RequestParam)를 인스턴스화 한다. `JSON 파싱`
    - 이 과정에서 `ArgumentResolver`가 **HttpMessageConverter**를 사용해 Request Message를 적절한 데이터 타입으로 변환 및 생성한다.
    ![image](https://github.com/user-attachments/assets/64561e13-83f2-4bd2-b630-9e3a2bf672bb)

### ArgumentResolver 상세 프로세스
1. Handler를 호출해 **파라미터 타입 정보**를 전달 받는다.
2. ArgumentResolver를 호출해 Controller의 파라미터 **객체 생성을 요청**한다.
    - `supportsParameter`: Parameter 처리 가능 여부 확인 + 실행 여부 반환
    - `resolveArgument`: 유입된 Parameter 가공
    ![image](https://github.com/user-attachments/assets/441128dd-cb90-47be-a5de-fc19b523644c)
    - Parameter 가공할 때, HTTPMessageConverter를 이용해 Request Message를 알맞은 데이터 타입으로 변환 → 생성 → 반환

### HTTPMessageConverter 상세 프로세스
HTTPMessageConverter`interface`를 이용해 Request Message를 알맞은 데이터 타입으로 변환 → 생성 → 반환한다.
- 기본 제공 Converter가 많고, JSON을 역직렬화할 수 있는 라이브러리 추가 시 자동으로 Conveter가 추가 등록된다.
- MessageConverter 구현체 중 MappingJacksonHttpMessageConverter(Spring Boot 기본 제공)를 사용하여 `리플렉션`을 통해 @RequestBody가 붙은 객체를 가져오고, Jackson의 ObjectMapper를 사용해 직렬화/역직렬화 후 적절한 객체 타입으로 Casting 된다. 


5. 모든 작업 완료 시, 리플렉션을 이용해 Controller를 호출하면서 파싱된 Request Message를 담고 있는 객체를 Paramter로 함께 전달한다.
![image](https://github.com/user-attachments/assets/a54f47a1-58bb-4f6f-8575-87268252a6d8)

# 유효성 검사; @Valid
Request Message의 값 여부 및 Format을 정의한 대로 검사해주는 어노테이션
### 실행 순서
ArgumentResolver#resolverArgument에서 실행+검사
1. Filter of Request
2. DispatcherServlet
3. HandlerMapping
4. HandlerAdapter
5. `HandlerInterceptor`
6. ArgumentResolver (HttpMessageConverter → validateIfApplicable)
7. Handler(Controller)
* Interceptor가 먼저 호출되고, ArgumentResolver가 실행된다.
![image](https://github.com/user-attachments/assets/2533615d-47ea-457c-8047-a53a9aa87d80)

## 공통 Response Message
### 1. Envelope Pattern
ApiResponse<T> 객체를 제네릭 타입으로 정의 후, Handler Method의 Return 타입으로 사용하는 방법
> **Controller의 Method를 정의할 때마다 항상 Return 타입을 ApiResponse<T>으로 감싸야 한다.**
![image](https://github.com/user-attachments/assets/0f7f21f0-dda7-4035-9d61-24a5ef25d97e)

### 2. ResponseBodyAdvice
ApiResponse<T> 객체를 정의하고, `AOP`를 통해 Handler가 Return하는 값을 중간에 가로채 정의한 타입으로 감싸는 방법
> **Method에서 다른 타입의 객체를 Return하면 AOP 구현체가 가로채서 ApiResponse<T> 객체로 감싸고 ApiResponse<T> 객체를 Return한다.**
![image](https://github.com/user-attachments/assets/83facb0c-9ea6-42f6-95bd-ff7d97b68691)

# Response 처리 프로세스; @Controller
![image](https://github.com/user-attachments/assets/1cf50bc8-ecd5-4a1e-9594-f7fd75ea528b)
1. 결과 값을 받은 Controller는 필요에 따라 Model 객체에 결과 값을 넣거나, View 정보를 담아 DispatcherServlet에게 보낸다.
2. DispatcherServlet은 View 정보를 ViewResolver에게 보낸다.  
ViewResolver는 해당 View를 찾아 DispatcherServlet에게 알려준다.
3. DispatcherServlet은 응답 할 View에게 Render를 지시하고, View는 응답 로직을 처리한다.
4. DispatcherServlet이 클라이언트에게 렌더링된 View를 전달한다.

# Response 처리 프로세스; @RestController
> **@Controller와 다른 점은?**  
> `ResponseBody 어노테이션의 유무`가 달라, 응답 프로세스 절차가 다르다.

![image](https://github.com/user-attachments/assets/9b7a8aea-05c4-4343-9138-1d73c010fe75)
![image](https://github.com/user-attachments/assets/6bcf1d6d-664e-43fc-8bf9-cfe73abdca39)
1. ViewResolver 대신 `Message Converter`가 실행되고, Return type에 맞는 Converter#canWrite만 실행된다.
    - @ResponseBody가 있어서 Message Converter가 실행되는 것
2. ControllerAdvice가 Return 값을 가로채고, 응답에 대한 공통 로직이 실행된다.
    - Handler의 Return type이 원시형일 경우, Type Casting 오류 발생
3. ReturnValueHandler가 공통 로직의 결과를 받아 Converter#writeInternal이 실행되면서 응답 메세지가 HTTP Response Body에 쓰여진다.
4. Client에게 HTTP Response가 전달된다.

# AOP ; `ResponseBodyAdvice`
각 Controller의 Method의 Return 값을 공통적으로 가공하고 싶은 요소가 있을 때 사용한다.
> Controller의 Method가 @ResponseBody 또는 @ResponseEntity로 Return 할 때 적용할 수 있다.

### 구조
![image](https://github.com/user-attachments/assets/3277baad-d752-4eb4-a1dd-c3ea78c58652)
- 특정 응답에 공통 처리 여부 판단
- Return type과 Converter type을 매개변수로 받아 원하는 대로 정의 가능

![image](https://github.com/user-attachments/assets/966f99aa-e286-4afa-877b-7280b000623b)
- Controller의 반환 데이터를 공통 처리하는 로직

# Error Handling 설계
### 공통 처리를 하는 이유
- 코드 가독성과 유지보수성
- AOP로 구현하는 것이 좋다
> 기본적으로 Spring Boot에서 제공하는 것을 최대한 사용하고, 필요에 따라 Custom해 표현하기.  
그러나 Custom은 잘못된 네이밍으로 부정확한 의미 전달과 기존 Exception이 의도와 다르게 사용될 수 있으므로 추천하지 않음

### 구현 `RestControllerAdvice`
![image](https://github.com/user-attachments/assets/14dd0d3e-dec2-4c6d-a6e3-1e1e4a03f4e3)