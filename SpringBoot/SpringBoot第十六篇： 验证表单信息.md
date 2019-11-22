## SpringBoot第十六篇： 验证表单信息

这篇文篇主要简述如何在springboot中验证表单信息。在springmvc工程中，需要检查表单信息，表单信息验证主要通过注解的形式。

## 构建工程

创建一个springboot工程，由于用到了 web 、thymeleaf、validator、el，引入相应的起步依赖和依赖，代码清单如下：

```xml
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
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-el</artifactId>
        </dependency>
    </dependencies>
```

## 创建一个PresonForm的Object类

```java
package com.forezp.entity;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
/**
 * Created by fangzhipeng on 2017/4/19.
 */
public class PersonForm {

    @NotNull
    @Size(min=2, max=30)
    private String name;

    @NotNull
    @Min(18)
    private Integer age;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String toString() {
        return "Person(Name: " + this.name + ", Age: " + this.age + ")";
    }
}
```

这个实体类，在2个属性:name,age.它们各自有验证的注解：

- @Size(min=2, max=30) name的长度为2-30个字符
- @NotNull 不为空
- @Min(18)age不能小于18

## 创建 web Controller

```java
@Controller
public class WebController extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/results").setViewName("results");
    }

    @GetMapping("/")
    public String showForm(PersonForm personForm) {
        return "form";
    }

    @PostMapping("/")
    public String checkPersonInfo(@Valid PersonForm personForm, BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            return "form";
        }

        return "redirect:/results";
    }
}
```

## 创建form表单

src/main/resources/templates/form.html:

```html
<html>
    <body>
        <form action="#" th:action="@{/}" th:object="${personForm}" method="post">
            <table>
                <tr>
                    <td>Name:</td>
                    <td><input type="text" th:field="*{name}" /></td>
                    <td th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Name Error</td>
                </tr>
                <tr>
                    <td>Age:</td>
                    <td><input type="text" th:field="*{age}" /></td>
                    <td th:if="${#fields.hasErrors('age')}" th:errors="*{age}">Age Error</td>
                </tr>
                <tr>
                    <td><button type="submit">Submit</button></td>
                </tr>
            </table>
        </form>
    </body>
</html>
```

## 注册成功的页面

src/main/resources/templates/results.html:

```html
<html>
    <body>
        Congratulations! You are old enough to sign up for this site.
    </body>
</html>
```

## 演示

启动工程，访问http://localhost:8080/：

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXgBoG7yqViaOu6kiaick04nhpLXESibxkVs6sUd1Ajibtia3Aglzmr2SMLbVqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果你输入A和15，点击 submit:

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXgKXwF5IJwsG0A1ZSuwM289HbdWwMWgWjHYygd40IpDEVCvvGxYtoyzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXg8Tx7jlU7r5yGZeKgBEzWhib7XGqyANPo6SiaMg48RfOuRtTiaX9xFibHhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



如果name 输入N, age为空：

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXgUr3lbLdDGAYsuRlOhAxPHZQDQicOT3FVvz796bsSRqiab2Z3UFEkhzsw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果输入：forezp. 18



## 参考资料

https://spring.io/guides/gs/validating-form-input/
