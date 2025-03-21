---
title: "Spring 이전에 자바 Servlet 알아보기"
date: 2023-02-15
update: 2023-02-15
tags:
  - Java
  - Servlet
series: "Java"
---
# Servlet
__자바 서블릿__ 은 스프링 MVC 아키텍처가 나오기 전 사용하였다. 
자바 서블릿은 자바를 사용하여 웹페이지를 동적으로 생성하는 서버측 프로그램을 말한다.

자바 서블릿은 웹 서버의 성능을 향상 하기 위해 사용되는 자바 클래스의 일종으로. WAS 서버를 쉽게 구축할 수 있도록 도와준다. 

<img src="https://user-images.githubusercontent.com/63226023/218739424-acd75b51-9c5c-4aae-8c80-b7c93e92acba.png">
> WAS (Web Application Server)이란
>
> -> WAS는 웹 애플리케이션을 통하여 필요한 기능을 수행하고 그 결과를 웹 서버에 전달해 준다.

즉 servlet은 정적인 리소스가 아닌 __클라이언트로 부터 요청을 받고 이에 대한 결과값을 처리하는__ 역할을 수행해 주는 자바 프로그램이다.


# Servlet 사용하기
자바 Servlet을 만들게 되면 `@WebServlet` 어노테이션으로 간단하게 url을 지정하고 이름 또한 지정해 줄 수 있다. Servlet은 `HttpServlet` 클래스를 상속 받아서 `service()`, `doGet()`, `doPost()` 메소드들을 오바라이딩 하여 request, response에 대한 클래스를 가져와 쉽게 HTTP 통신을 지원한다.

```java
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


@WebServlet(name = "hello", urlPatterns = { "/hello" })
public class HelloServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	@Override
	protected void service(HttpServletRequest req, HttpServletResponse resp) 
		throws ServletException, IOException {
		super.service(req, resp);
	}

	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response) 
		throws ServletException, IOException {
		// TODO Auto-generated method stub
		response.getWriter().append("Served at: ").append(request.getContextPath());
	}

	@Override
	protected void doPost(HttpServletRequest request, HttpServletResponse response) 
		throws ServletException, IOException {
		// TODO Auto-generated method stub
		doGet(request, response);
	}
}
```

주요 메소드를 살펴보면 다음과 같다.
> 1. init()
> - Servlet이 메모리에 로드 될 때 한번 호출된다 (코드가 수정된다면 다시 호출)
> 2. doGet()
> - GET 방식으로 data전송 시 호출된다.
> 3. doPost()
> - POST 방식으로 data전송 시 호출된다.
> 4. service()
> - 모든 요청은 service()를 통하여 doXXX()관련 메소드로 이동한다.
> 5. destroy()
> - Servlet이 메모레에서 해제되면 호출된다.

## Hello 요청
Servlet의 doGet() 메소드를 이용하면 HTTP GET 요청을 처리하여 "Hello" 라는 메시지를 생성하고 HTML 형식으로 응답을 작성하여 전송할 수 있다.
```java
public void doGet(HttpServletRequest request, HttpServletResponse response)
		throws ServletException, IOException {
	response.setContentType("text/html");

	PrintWriter out = response.getWriter();
	out.println("<html>");
	out.println("<head><title>Hello World Servlet</title></head>");
	out.println("<body>");
	out.println("<h1>Hello</h1>");
	out.println("</body></html>");
}
```
html 파일을 직접 생성하지 않고 이러한 자바 코드안에 지정하여 사용할 수 있지만 이는 굉장히 코드 작성시 불편함이 있다. 이러한 불편함을 해소 하기 위해 __JSP (Java Server Page)__ 가 나오게 되었다.

# JSP
Servlet은 자바 코드안에 HTML이 있지만 JSP는 HTML 내에 자바 코드를 삽입하여 웹 서버에서 동적으로 웹 페이지를 생성하여 웹 브라우저에 돌려주는 언어이다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%!
	String name;

	public void init() {
		name = "저는 JSP 입니다";
	}
%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>
<body>
	<%
		out.print(name);
	%>
</body>
</html>
```

JSP의 문법은 `<% %>` 에 대한 블록을 지정하여 내부에 자바 코드를 구현 할 수 있고 많은 문법들은 인터넷에 자료들을 쉽게 볼 찾아 볼 수 있다.

이렇게 JSP와 함께 Web Application Architecture의 구성은 처음에는 model1의 구조로 사용이 되었다.

model1 구조는 view와 logic을 JSP 페이지 하나에서 처리하는 구조로 HTML, 자바, 비지니스 로직, JDBC 연결코드 등 이 한 곳에 들어가 있어 굉장히 유지보수가 어려워진다.😨 

이에 따라 __model2(MVC 패턴) 구조__ 가 나오게 되었다.

# MVC 패턴
model2의 구조는 모든 처리를 JSP가 아닌 `클라이언트의 요청에 대한 처리는 Servlet`이, `logic처리는 java class(Service, Dao..)`, `클라이언트에게 출력하는 response page(view)는 JSP`가 담당하여 개발한다.

model2 구조를 흔히 MVC(Model, View, Controller) 패턴이라고 부르게 되는데 Servlet이 Controller의 역할을 맡고, JSP가 View로 사용되어 서로 Model에 대한 데이터를 참조하게 된다.
```java
@WebServlet("/test")
public class SomeJspPage extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
		throws ServletException, IOException {
        String path = "/bookinfo.jsp";
        // 어떠한 book에 대한 model을 넘긴다고 하자
        BookDto book = new BookDto(생성자 세팅)

        // model에 book 데이터 전달
        request.setAttribute("bookinfo", book);

        RequestDispatcher dispatcher = request.getRequestDispatcher(path);
        dispatcher.forward(request, response);
    }
}
```
`RequestDispatcher` 클래스를 사용하여 요청을 JSP/Servlet 내에서 원하는 자원으로 요청을 넘기거나, 처리를 요청하고 결과를 얻어오는 기능을 수행해 주는 클래스이다. 위의 예시는 MVC 패턴으로 `forward()` 메소드를 통해여 model(bookinfo)과 함께 bookinfo.jsp로 이동하게 해준다. 

또한 요청을 받은 페이지에서는 request.getAttribute()로 해당하는 데이터를 가져와 동적으로 화면을 보여줄 수 있다.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8" import="com.ssafy.move.BookDto"%>
<!DOCTYPE html>
<html>
<head>
<%
	BookDto bookDto = (BookDto) request.getAttribute("bookinfo");
	if (bookDto == null) {
%>
<script>
	alert("Book 정보가 없습니다.");
</script>
<%
	} else {
%>

<meta charset="UTF-8">
<title>책정보(book.jsp)</title>
</head>
<body>
	<div>
		<h3>
			[<%=bookDto.getBookName()%>] 정보
		</h3>
		ISBN :
		<%=bookDto.getIsbn()%><br> 출판사 :
		<%=bookDto.getPublisher()%><br> 가격 :
		<%=bookDto.getPrice()%>원<br>
	</div>
	<%
		}
	%>
</body>
</html>
```

`dispatcher.forward()` 메소드 이외에 redirect의 메소드가 있는데 둘의 차이점은 다음과 같다.
> forward()
> - forward 메소드는 현재 서블릿에서 다른 서블릿/JSP 페이지로 요청을 전달 할 수 있다.
> redirect()
> - redirect 메소드는 새로운 요청을 생성하고, 새로운 URL로 이동을 하게된다. 따라서 클라이언트에서는 두 번의 요청이 발생하게 되고, 이에 따라 URL도 변경된다.
> - redirect는 따라서 새로운 요청을 생성하므로, 기존의 요청 객체(bookinfo)에 저장된 데이터는 새로운 요청 객체에서 사용할 수 없다.

