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