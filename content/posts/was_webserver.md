---
title: "[Web] Web Server와 WAS"
date: 2021-09-27T14:34:24+09:00
authors: ["Jungminsayho"]
# tags: [""] 
# draft: true

---

<br><br>


## WAS (Web Application Server)
클라이언트로부터 웹 서버가 요청을 받으면 애플리케이션에 대한 로직을 실행하여 서버로 다시 반환해주는 소프트웨어이다.<br>
웹 서버와 DBMS 사이에서 동작하는 미들웨어로써, 컨테이너 기반으로 동작한다

<br>

<img src="https://images.velog.io/images/jungminnn/post/c735c51e-3c3f-49ff-a8f7-94861cec80d1/image.png" width="100%" height="100%"/>


<br><br>

#### 1. HTTP 서버 (웹 서버)

URL 주소의 해석을 맡아 주는 HTTP서버를 말한다.<br>
단순히 어떤 주소(URL) 요청이 들어왔을 경우, 그 주소에 미리 매핑되어 있는 콘텐츠(HTML 파일이나 이미지 등) 즉, `정적 컨텐츠`를 사용자의 브라우저에 응답 형태로 전송하는 역할을 한다.<br><br>
 이 때 만일 요청된 URL이 서블릿 클래스 또는 JSP파일(ex. http://www.sevlet.com/servlet 또는 http://www.wervlet.com/home.jsp)일 경우 HTTP 서버는 이를 웹 컨테이너에서 처리하도록 클라이언트의 요청을 넘겨준다.

<br>

#### 2. 웹 컨테이너(Web Container)

서블릿 클래스 또는 JSP 파일, 즉 `동적 컨텐츠`의 실행 요청을 처리한다.<br> 
요청된 URL에 맞는(미리 설정된) 서블릿 클래스 또는 JSP 파일을 실행하여 그 결과를 HTTP 서버에 넘겨주게 되고, 이는 응답 메시지의 형태로 사용자의 브라우저에 전송된다.

<br><br>

## 웹서버와 WAS의 차이점
`웹 컨테이너의 유무`로 웹서버와 WAS를 나눌 수 있고,
둘의 가장 큰 차이점은 요청을 받아 처리하는 컨텐츠이다.<br>
`웹서버`(ex. `Apache`, `Nginx`)의 경우 정적인 컨텐츠 (html, css, image 등)를 요청받아 처리하고, <br>
`WAS`(ex. `Tomcat`)의 경우 개발 언어를 읽고, 동적인 컨텐츠 (JSP, ASP, PHP 등)를 요청받아 처리한다. 

<br><br>

## 웹서버와 WAS를 나눠야 하는 이유
`WAS`의 경우 `웹서버 + 웹 컨테이너`의 개념이라, 웹서버가 없더라도 웹서버의 역할을 동시에 수행 가능하지만, 굳이 둘을 나눠서 사용하는 이유가 있다.

<br>

> 사용자가 많지 않거나 트래픽이 적을 때는, WAS 단독 사용이 효율적이며 개발 및 테스트 시스템 구성 시 활용 가치가 높다.

<br>

#### 1. 데이터 처리 방식
둘은 처리하는 `데이터의 형태`가 다르다.<br>
굳이 빠른 시간에 처리할 수 있는 정적데이터를 WAS에서 처리하여 부하를 줄 필요가 없다.
웹서버와 WAS의 기능적 분류를 통해 효과적인 분산을 유도한다.

WAS는 동적 처리에 최적화되어있는 서비스이기 때문에, WAS가 정적, 동적 처리를 같이 할 경우 효율이 좋지 않다.

<br>

#### 2. 보안
WAS의 경우 DB서버에 대한 접속 정보가 있기 때문에, 외부로 노출될 경우 보안상 문제가 될 수 있다.<br>
그래서 웹서버를 앞단에 위치시켜 보안을 유지한다.
**(`웹 서버` - DMZ 구간, `WAS` - 내부망)**

<br>

<img src="https://images.velog.io/images/jungminnn/post/c6a300d6-f303-4e2f-9ddd-70e5f2038616/image.png" width="100%" height="100%"/>


<br><br>

### References<br>
https://goldsony.tistory.com/37<br>
https://yang1650.tistory.com/8<br>
https://oasisfores.com/rest-api-server-apache-flask-wsgi/<br>
https://helloworld-88.tistory.com/71
