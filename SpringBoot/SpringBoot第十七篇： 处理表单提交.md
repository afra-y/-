## SpringBoot第十七篇： 处理表单提交

这篇文件主要介绍通过springboot 去创建和提交一个表单。

## 创建工程

涉及了 web，加上spring-boot-starter-web和spring-boot-starter-thymeleaf的起步依赖。

```java
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-thymeleaf</artifactId>
            </dependency>

    </dependencies>
```

## 创建实体

代码清单如下：

```java
public class Greeting {

    private long id;
    private String content;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

}
```

## 创建Controller

```java
@Controller
public class GreetingController {

    @GetMapping("/greeting")
    public String greetingForm(Model model) {
        model.addAttribute("greeting", new Greeting());
        return "greeting";
    }

    @PostMapping("/greeting")
    public String greetingSubmit(@ModelAttribute Greeting greeting) {
        return "result";
    }

}
```

## 页面展示层

src/main/resources/templates/greeting.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <h1>Form</h1>
    <form action="#" th:action="@{/greeting}" th:object="${greeting}" method="post">
        <p>Id: <input type="text" th:field="*{id}" /></p>
        <p>Message: <input type="text" th:field="*{content}" /></p>
        <p><input type="submit" value="Submit" /> <input type="reset" value="Reset" /></p>
    </form>
</body>
</html>
```

src/main/resources/templates/result.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <h1>Result</h1>
    <p th:text="'id: ' + ${greeting.id}" />
    <p th:text="'content: ' + ${greeting.content}" />
    <a href="/greeting">Submit another message</a>
</body>
</html>
```

启动工程，访问ttp://localhost:8080/greeting:

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXg502t7IKWHqV9syTWpI1ojrCYU1NicOF6Chsgy8pZU3Vibb4S0LTgQbzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

点击submit:

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXgaqVUicGtWgdlCZbx0ibRwGSPw24usdJvuic3pB9NcLFJAteaxW369LakA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 参考资料

https://spring.io/guides/gs/handling-form-submission/