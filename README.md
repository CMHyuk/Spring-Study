## 스프링 MVC 구조
클라이언트로부터 요청이 들어오면 DispatcherServlet이 요청을 받습니다.

DispatcherServlet은 HandlerMapping을 통해 요청 URL에 매핑된 핸들러를 조회합니다.

이후 핸들러를 실행할 수 있는 핸들러 어댑터를 조회하고 실행해 핸들러를 실행합니다.

핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환합니다.

그리고 viewResolver를 찾고 실행하고 view를 반환합니다.



1. **핸들러 조회**: 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.


2. **핸들러 어댑터 조회**: 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.


4. **핸들러 어댑터 실행**: 핸들러 어댑터를 실행한다.


5. **핸들러 실행**: 핸들러 어댑터가 실제 핸들러를 실행한다.


6. **ModelAndView 반환**: 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 **변환**해서 반환한다.


7. **viewResolver 호출**: 뷰 리졸버를 찾고 실행한다.  
   - JSP의 경우: InternalResourceViewResolver 가 자동 등록되고, 사용된다.  
   - JSON의 경우 : viewResolver가 아닌 **MappingJackson2HttpMessageConverter** 동작


8. **View반환** : 뷰리졸버는 뷰의 논리이름을 물리이름으로 바꾸고,렌더링 역할을 담당하는 뷰객체를 반환한다.  
   - JSP의 경우 InternalResourceView(JstlView) 를 반환하는데, 내부에 forward() 로직이 있다.


9. **뷰렌더링** : 뷰를 통해서 뷰를 렌더링한다.

~~~
HandlerMapping

0 = RequestMappingHandlerMapping 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = BeanNameUrlHandlerMapping (스프링 빈의 이름으로 찾음)
~~~  

~~~
HandlerAdapter

0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
2 = SimpleControllerHandlerAdapter : Controller 인터페이스
~~~  

~~~
ViewResolver

1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환한다. (예: 엑셀 파일 생성
기능에 사용)
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.
~~~

## 스프링 빈 생명주기  
스프링 컨테이너 생성 → 스프링 빈 생성 → 의존 관계 주입 → 초기화 콜백 → 사용 → 소멸 전 콜백 → 종료 

## MessageConverter
스프링 메시지 컨버터는 HTTP 요청 또는 응답의 바디를 객체로 변환하거나 객체를 HTTP 요청 또는 응답의 바디로 변환하는 기능을 수행합니다.

핸들러 어댑터가 ReqeustMappingHandlerAdapter일 경우 ArgumentResolver 
@Requestparam @ModelAtttibrute등 여러 가지 애노테이션을 처리 해줍니다.

이때 @ReuqestBody와 같이 Http 메시지를 처리하는 애노테이션의 경우 HttpMessageConverter가 동작합니다.
내부적으로 http 요청 데이터를 읽기 위해 canRead()메서드가 실행되는데, 대상 클래스 타입을 지원하는지? Content-Type 미디어 타입을 지원하는지?
확인 후 조건 만족 시 read()를 호출해 객체를 생성 후 반환합니다.

응답 데이터의 경우 ArgumentResolver가 아닌 ReturnValueHandler가 HttpMessageConverter를 호출해 
canWrite()가 호출되어 요청 데이터와 마찬가지로 클래스 타입, 미디어 타입을 지원하는지 확인 후 조건 만족 시 write() 메서드를 호출해 http 응답 메시지 바디에 데이터를 생성합니다.

~~~
메시지 컨버터 종류

0 = ByteArrayHttpMessageConverter : byte[] 데이터를 처리
1 = StringHttpMessageConverter : String 문자로 데이터를 처리
2 = MappingJackson2HttpMessageConverter : application/json
~~~

## @ExceptionHandler
클라이언트에서 HTTP 요청을 서버에 보내면, 스프링의 디스패처 서블릿이 이를 받아 해당 요청을 처리할 컨트롤러를 선택합니다. 컨트롤러는 요청을 처리하고 결과를 반환합니다. 이때, 컨트롤러에서 예외가 발생하면, @ExceptionHandler 어노테이션이 지정된 메서드가 해당 예외를 처리하게 됩니다.

## 서블릿 필터 vs 인터셉터

서블릿 필터는 웹 애플리케이션 전역에 적용되며, 모든 요청에 대해 처리를 수행합니다.

스프링 인터셉터는 스프링 애플리케이션에서만 사용 가능하며, 핸들러 메서드를 처리하는 컨트롤러에서만 실행됩니다.

서블릿 필터 : 보안, 로깅, 인증 및 권한 부여와 같은 일반적인 작업을 수행하는 데 사용됩니다.

스프링 인터셉터 :  요청 처리 중에 로그인 사용자 검증, 권한 검사, 세션 처리 및 캐싱과 같은 작업에 사용됩니다.

* 정상 흐름

preHandle : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.)
preHandle 의 응답값이 true 이면 다음으로 진행하고, false 이면 더는 진행하지 않는다. false

인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다.
postHandle : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)
afterCompletion : 뷰가 렌더링 된 이후에 호출된다.

* 오류 발생

preHandle : 컨트롤러 호출 전에 호출된다.
postHandle : 컨트롤러에서 예외가 발생하면 postHandle 은 호출되지 않는다.
afterCompletion : afterCompletion 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤
예외가 발생했는지 로그로 출력할 수 있습니다.

예외가 발생하면 postHandle() 는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면
afterCompletion() 을 사용해야 합니다.
예외가 발생하면 afterCompletion() 에 예외 정보( ex )를 포함해서 호출됩니다.

## @Transactional
Spring Framework에서 @Transactional 어노테이션은 메소드나 클래스에 부여되어, 해당 메소드나 클래스에서 수행되는 모든 데이터베이스 연산이 하나의 트랜잭션으로 묶이도록 하는 어노테이션입니다.
@Transactional 어노테이션이 부여된 메소드가 호출될 때, Spring은 트랜잭션을 시작하고 메소드 수행 도중에 예외가 발생하지 않으면 트랜잭션을 커밋하고, 예외가 발생하면 트랜잭션을 롤백합니다.

Spring에서 @Transactional 어노테이션은 여러 개의 옵션을 가지고 있습니다. 그 중 일부 옵션은 아래와 같습니다.

* isolation : 트랜잭션 격리 수준을 설정할 수 있습니다. 기본적으로 READ_COMMITTED 수준을 사용합니다.
* propagation : 트랜잭션 전파 규칙을 설정할 수 있습니다. 이전 트랜잭션이 이미 시작되어 있는 경우, 새로운 트랜잭션을 시작하거나 기존 트랜잭션을 사용하는 등의 설정을 할 수 있습니다.
* readOnly : 트랜잭션이 읽기 전용인 경우 true로 설정할 수 있습니다. 이 경우 데이터베이스에서 성능을 개선할 수 있습니다.
* rollbackFor : 특정 예외가 발생하면 트랜잭션을 롤백하도록 설정할 수 있습니다.
* noRollbackFor : 특정 예외가 발생하더라도 트랜잭션을 롤백하지 않도록 설정할 수 있습니다.