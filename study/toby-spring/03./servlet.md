# Servlet

WAS는 클라이언트와 서버 간의 소켓 통신에 필요한 TCP/IP 연결 관리와 HTTP 프로토콜 해석 등의 네트워크 기반 작업을 추상화해 일종의 실행 환경을 제공한다. 따라서 웹 프로그래머는 TCP/IP 연결을 직접 생성하고 HTTP 프로토콜을 해석하는 과정을 생략해 웹을 쉡게 구현할 수 있다. 결록 관련 기술을 깊이 알지 못한 채로 WAS가 제공하는 추상화되 API로 네트워크에 접근한다.&#x20;

그러나 웹 기반 서비스의 영역은 지속적으로 넓어지고 있기에 단순히 사용하는데서 그쳐서는 안된다. WAS의 내부구조와 동작 원리를 이해하지 못하면 고성능, 고가용성에 대한 요구를 충족시길 수 없다.&#x20;

{% embed url="https://yangbongsoo.gitbook.io/study/servlet_container" %}

## 서블릿이란

javax.servlet.Servlet 인터페이스를 구현한 것이 서블릿이다. 서블릿은 서블릿 컨테이너 내에 등록된 후 서블릿 컨테이너에 의해 성성, 호출, 소멸이 이뤄진다. 다시 말해 서블릿은 독립적으로 실행되지 않고 main 메서드가 필요하지 않으며 서블릿 컨테이너에 의해 서블릿의 상태가 바뀐다.&#x20;



### GenericServlet

서블릿 인터페이스만 구현하면 서블릿 컨테이너가 생성, 소멸 등의 라이프 사이클을 관리해 준다. 그러나 서블릿 인터페이스에서 구현 필요한 메서드들을 매번 구현할 수 없으므로, 구현된 GenericServlet이란 추상 클래스를 제공한다. 해당 메서드를 상속받아 메인 로직인 service 메서드만 구현하면된다.

```java
public abstract class GenericServlet implements Servlet, ServletConfig,
        java.io.Serializable {

  @Override
    public abstract void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException;

}
```

### HttpServlet

일반적으로 서블릿이라 하면 거의 HttpServlet 을 상속받은 서블릿을 의미한다. GenericServlet의 유일한 추상 메서드인 service 를 HTTP 프로토콜 요청 메서드에 적합하게 재구현한 것이다.&#x20;

서블릿 컨테이너는 받은 요청에 대해 서블릿을 선택한 후 servlet 인터페이스에 정의된 service(ServletRequest, ServletResponse)를 호출한다. 그러면 클래스 상속 위계에 따라 그 처리가 부모 클래스인 GenericServlet에서 자식 클래스인 HttpServlet으로 넘어온다. HttpServlet의 service 메서드 내에서는 HTTP 요청 메서드에 의해 여러 doXXX 메서드로 분기돼 처리된다.

<pre class="language-java"><code class="lang-java">public abstract class HttpServlet extends GenericServlet {
<strong>    protected void service(HttpServletRequest req, HttpServletResponse resp)
</strong>            throws ServletException, IOException {
        String method = req.getMethod();
        
        if (method.equals(METHOD_GET)) {
            ...
        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);

        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);

        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);

        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);

        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);

        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);

    }    
}
</code></pre>

| ►서버 TCP/IP 연결 대기, 소켓 연결                                 |
| ------------------------------------------------------- |
| ► HTTP 요청 메시지 파싱                                        |
| ► HTTP method, URL 인지                                   |
| ► Content-Type 확인                                       |
| ► HTTP 메시지 바디 내용 파싱                                     |
| ► 저장 프로세스 실행                                            |
| <mark style="background-color:red;">► 비지니스 로직 실행</mark> |
| ► HTTP 응답 메시지 생성                                        |
| ► TCP/IP에 응답 전달, 소켓 종료                                  |

서블릿은 개발자가 비지니스 로직에 집중할 수 있도록, HTTP 통신을 위한 일련의 과정들을 모두 자동으로 해준다. 그덕에 개발자는 단순히 HTTP 요청 메시지로 생성된 request를 읽어서 비즈니스 로직을 수행하고 response를 반화하면 된다.&#x20;



서블릿이 요구하는 규약을 지키면서 개발을 진행하면, http 요청 정보 사용과 응답값 생성이 쉽고 빨라진다.&#x20;

## 서블릿 컨테이너와 서블릿 동작 방식



## 프론트 컨트롤러 패턴



## Dispatcher Servlet 의 요청 처리 과정





## 스프링 컨테이너 맛보기







{% embed url="https://steady-coding.tistory.com/462#recentComments" %}

{% embed url="https://jusungpark.tistory.com/15" %}
