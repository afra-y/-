## Spring Boot第三篇：SpringBoot 整合JPA

JPA全称Java Persistence API.JPA通过JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。

JPA 的目标之一是制定一个可以由很多供应商实现的API，并且开发人员可以编码来实现该API，而不是使用私有供应商特有的API。

JPA是需要Provider来实现其功能的，Hibernate就是JPA Provider中很强的一个，应该说无人能出其右。从功能上来说，JPA就是Hibernate功能的一个子集。

## 添加相关依赖

添加spring-boot-starter-jdbc依赖：

```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa
            </artifactId>
        </dependency>
```

添加mysql连接类和连接池类：

```java
    <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

## 配置数据源，在application.properties文件配置：

```java
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8
    username: root
    password: 123456

  jpa:
    hibernate:
      ddl-auto: update  # 第一次简表create  后面用update
    show-sql: true
```

注意，如果通过jpa在数据库中建表，将jpa.hibernate,ddl-auto改为create，建完表之后，要改为update,要不然每次重启工程会删除表并新建。

## 创建实体类

通过@Entity 表明是一个映射的实体类，  @Id表明id， @GeneratedValue 字段自动生成

```java
@Entity
public class Account {
    @Id
    @GeneratedValue
    private int id ;
    private String name ;
    private double money;

...  省略getter setter
}
```

## Dao层

数据访问层，通过编写一个继承自 JpaRepository 的接口就能完成数据访问,其中包含了几本的单表查询的方法，非常的方便。值得注意的是，这个Account 对象名，而不是具体的表名，另外Interger是主键的类型，一般为Integer或者Long

```java
public interface AccountDao  extends JpaRepository<Account,Integer> {
}
```

## Web层

在这个栗子中我简略了service层的书写，在实际开发中，不可省略。新写一个controller，写几个restful api来测试数据的访问。

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @Autowired
    AccountDao accountDao;

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public List<Account> getAccounts() {
        return accountDao.findAll();
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public Account getAccountById(@PathVariable("id") int id) {
        return accountDao.findOne(id);
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    public String updateAccount(@PathVariable("id") int id, @RequestParam(value = "name", required = true) String name,
                                @RequestParam(value = "money", required = true) double money) {
        Account account = new Account();
        account.setMoney(money);
        account.setName(name);
        account.setId(id);
        Account account1 = accountDao.saveAndFlush(account);

        return account1.toString();

    }

    @RequestMapping(value = "", method = RequestMethod.POST)
    public String postAccount(@RequestParam(value = "name") String name,
                              @RequestParam(value = "money") double money) {
        Account account = new Account();
        account.setMoney(money);
        account.setName(name);
        Account account1 = accountDao.save(account);
        return account1.toString();

    }

}
```

通过postman请求测试，代码已经全部通过测试。

源码下载：https://github.com/forezp/SpringBootLearning

完整的例子：

```java
package com.sitech.demo.entity;

import javax.persistence.*;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
    private int age;
    private String sex;

    @Override
    public String toString(){
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                '}';
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public String getSex() {
        return sex;
    }
}

```

```java
package com.sitech.demo.dao;

import com.sitech.demo.entity.Student;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface StudentDao extends JpaRepository<Student,Integer> {

     Student findStuById(Integer id);
    @Query(name = "findStuById",nativeQuery = true,value = "select * from student where name=:name ")
   	List<Student> findStuByName(@Param("name") String name);

    @Query(name = "updateById",nativeQuery = true,value = "select * from student where id=:id ")
    Student updateById(Integer id);

}

```

```java
package com.sitech.demo.service;

import com.sitech.demo.entity.Student;
import org.springframework.data.domain.Page;

import java.util.List;

public interface StudentService {
    Student save(Student student);
    Student update(Student student);
    void delete(Integer id);
    Student findStuById(Integer id);
    List<Student> findStuByName(String name);
    Page<Student> findAll(int page,int pageSize);
    Student updateById(int i);
}

```

```java
package com.sitech.demo.service;

import com.sitech.demo.dao.StudentDao;
import com.sitech.demo.entity.Student;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

import java.util.List;
@Service
public class StudentServiceImpl implements StudentService {
    @Autowired
    private StudentDao studentDao;
    @Override
    public Student save(Student student) {
        return studentDao.save(student);
    }

    @Override
    public Student update(Student student) {
        return studentDao.save(student);
    }

    @Override
    public void delete(Integer id) {
        studentDao.deleteById(id);
    }

    @Override
    public Student findStuById(Integer id) {
        return studentDao.findStuById(id);
    }

    @Override
    public List<Student> findStuByName(String name) {
        return studentDao.findStuByName(name);
    }

    @Override
    public Page<Student> findAll(int page, int pageSize) {
        Pageable pageable = PageRequest.of(page,pageSize);
        return studentDao.findAll(pageable);
    }

    @Override
    public Student updateById(int  i){
        return studentDao.updateById(i);
    }

}

```

```java
package com.sitech.demo.controller;

import com.sitech.demo.entity.Student;
import com.sitech.demo.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletResponse;
import java.util.List;
@RestController
@RequestMapping("/s")
public class StudentController {
    @Autowired
    private StudentService studentService;

    @PostMapping("/add")
    public Student save(Student student){
        return studentService.save(student);
    }
    @PostMapping("/update")
    public Student update(Student student){
        return studentService.save(student);
    }
    @GetMapping("/del/{id}")
    public String del(@PathVariable Integer id){
        studentService.delete(id);
        return "ojbk";
    }
    @GetMapping("findByName/{name}")
    public List<Student> findStuByName(@PathVariable String name)
    {
        return studentService.findStuByName(name);
    }
    @GetMapping("/query")
    public Page<Student> findByPage(Integer page, HttpServletResponse response){
        response.setHeader("Access-Control-Allow-Origin","*");
        if (page==null || page<=0){
            page =0;
        }else {
            page-=1;
        }
        return studentService.findAll(page,5);
    }


    @GetMapping("/update/{id}")
    public Student update(@PathVariable int id){

        return studentService.updateById(id);
}

}

```

