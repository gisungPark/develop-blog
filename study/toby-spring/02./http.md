# Http 기본

웹 요청과 응답의 기본 구조를 이해하고 내용을 확인할 수 있어야  한다.

* Request
  * Request Line: Method, Path, HTTP Version
  * Headers
  * Message Body
* Response
  * Status Line: HTTP version, Status Code, Status Text
  * Headers
  * Message Body

```bash
GET /hello?name=Spring HTTP/1.1
Accept: */*                        -- 모든 타입을 다 받겠다.
Accept-Encoding: gzip, deflate
Connection: keep-alive
Host: localhost:8080
User-Agent: HTTPie/3.2.2



HTTP/1.1 200 
Connection: keep-alive
Content-Length: 14
Content-Type: text/plain;charset=ISO-8859-1
Date: Sun, 23 Jul 2023 12:59:05 GMT
Keep-Alive: timeout=60

```

