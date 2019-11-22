## SpringBoot第十篇：springboot集成swagger2，构建优雅的Restful API

swagger,中文“拽”的意思。它是一个功能强大的api框架，它的集成非常简单，不仅提供了在线文档的查阅，而且还提供了在线文档的测试。另外swagger很容易构建restful风格的api，简单优雅帅气，正如它的名字。

一、引入依赖

```xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.6.1</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.6.1</version>
        </dependency>
```

二、写配置类

```java
@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.forezp.controller"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("springboot利用swagger构建api文档")
                .description("简单优雅的restfun风格，http://blog.csdn.net/forezp")
                .termsOfServiceUrl("http://blog.csdn.net/forezp")
                .version("1.0")
                .build();
    }
}
```

通过@Configuration注解，表明它是一个配置类，@EnableSwagger2开启swagger2。apiINfo()配置一些基本的信息。apis()指定扫描的包会生成文档。

三、写生产文档的注解

swagger通过注解表明该接口会生成文档，包括接口名、请求方法、参数、返回信息的等等。

- @Api：修饰整个类，描述Controller的作用
- @ApiOperation：描述一个类的一个方法，或者说一个接口
- @ApiParam：单个参数描述
- @ApiModel：用对象来接收参数
- @ApiProperty：用对象接收参数时，描述对象的一个字段
- @ApiResponse：HTTP响应其中1个描述
- @ApiResponses：HTTP响应整体描述
- @ApiIgnore：使用该注解忽略这个API
- @ApiError ：发生错误返回的信息
- @ApiParamImplicitL：一个请求参数
- @ApiParamsImplicit 多个请求参数

现在通过一个栗子来说明：

```java
package com.forezp.controller;

import com.forezp.entity.Book;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.*;
import springfox.documentation.annotations.ApiIgnore;

import java.util.*;

/**
 * 用户创建某本图书    POST    /books/
 * 用户修改对某本图书    PUT    /books/:id/
 * 用户删除对某本图书    DELETE    /books/:id/
 * 用户获取所有的图书 GET /books
 *  用户获取某一图书  GET /Books/:id
 * Created by fangzhipeng on 2017/4/17.
 * 官方文档：http://swagger.io/docs/specification/api-host-and-base-path/
 */
@RestController
@RequestMapping(value = "/books")
public class BookContrller {

    Map<Long, Book> books = Collections.synchronizedMap(new HashMap<Long, Book>());

    @ApiOperation(value="获取图书列表", notes="获取图书列表")
    @RequestMapping(value={""}, method= RequestMethod.GET)
    public List<Book> getBook() {
        List<Book> book = new ArrayList<>(books.values());
        return book;
    }

    @ApiOperation(value="创建图书", notes="创建图书")
    @ApiImplicitParam(name = "book", value = "图书详细实体", required = true, dataType = "Book")
    @RequestMapping(value="", method=RequestMethod.POST)
    public String postBook(@RequestBody Book book) {
        books.put(book.getId(), book);
        return "success";
    }
    @ApiOperation(value="获图书细信息", notes="根据url的id来获取详细信息")
    @ApiImplicitParam(name = "id", value = "ID", required = true, dataType = "Long",paramType = "path")
    @RequestMapping(value="/{id}", method=RequestMethod.GET)
    public Book getBook(@PathVariable Long id) {
        return books.get(id);
    }

    @ApiOperation(value="更新信息", notes="根据url的id来指定更新图书信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "图书ID", required = true, dataType = "Long",paramType = "path"),
            @ApiImplicitParam(name = "book", value = "图书实体book", required = true, dataType = "Book")
    })
    @RequestMapping(value="/{id}", method= RequestMethod.PUT)
    public String putUser(@PathVariable Long id, @RequestBody Book book) {
        Book book1 = books.get(id);
        book1.setName(book.getName());
        book1.setPrice(book.getPrice());
        books.put(id, book1);
        return "success";
    }
    @ApiOperation(value="删除图书", notes="根据url的id来指定删除图书")
    @ApiImplicitParam(name = "id", value = "图书ID", required = true, dataType = "Long",paramType = "path")
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
    public String deleteUser(@PathVariable Long id) {
        books.remove(id);
        return "success";
    }

    @ApiIgnore//使用该注解忽略这个API
    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    public String  jsonTest() {
        return " hi you!";
    }
}
```

通过相关注解，就可以让swagger2生成相应的文档。如果你不需要某接口生成文档，只需要在加@ApiIgnore注解即可。需要说明的是，如果请求参数在url上，@ApiImplicitParam 上加paramType = “path” 。

启动工程，访问：http://localhost:8080/swagger-ui.html ，就看到swagger-ui:

![img](https://mmbiz.qpic.cn/mmbiz_png/rtJ5LhxxzwndECAOic552dTLavibQTDicXg6vbGiaqpPHI1mXeTCQQic4QmVFtdPX5ibp69XJ9ibn4fI0JgJ7uTVlOYFg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

整个集成过程非常简单，但是我看了相关的资料，swagger没有做安全方面的防护，可能需要我们自己做相关的工作。

### 四、参考资料

swagger.io

Spring Boot中使用Swagger2构建强大的RESTful API文档

实际开发例子：

swagger部分代码：

```java
package com.sitech.salecenter.salecenterservice.config.swagger2;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @author qianym
 * @create 2018/5/22
 * @since 3.0
 */
@Configuration
@EnableSwagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.sitech.salecenter.salecenterservice.service"))
                .paths(PathSelectors.regex("/.*"))
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("美国联通Restful API")
                .description("美国联通项目Restful API")
                .version("1.0")
                .build();
    }
}
```

删除的入参（其他部分的出参太过于庞大）

```java
package com.sitech.salecenter.salecenterservice.dto.in;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModelProperty;

public class ProspectiveCustInDel {
    @JsonProperty("LEAD_ID")
    @ApiModelProperty(name ="LEAD_ID", value = "潜在用户id")
    private String lead_id;
    
    @JsonProperty("ADDRESS_ID")
    @ApiModelProperty(name ="ADDRESS_ID", value = "潜在用户地址id")
    private String address_id;
	//相关的get set方法
}
```

```java

package com.sitech.salecenter.salecenterservice.dto.out;

import com.fasterxml.jackson.annotation.JsonProperty;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.util.Date;

/**
 * @author yu
 */
@ApiModel(description = "潜在客户信息")
public class ProspectiveCustOut {
        @JsonProperty("LEAD_ID")
        @ApiModelProperty(value = "潜在客户标识")
        private String lead_id;
        @JsonProperty("STATUS")
        @ApiModelProperty(value = "潜在客户状态")
        private String status;
        @JsonProperty("END_ID")
        @ApiModelProperty(value = "企业编码")
        private String end_id;
        @JsonProperty("LEAD_OWNER")
        @ApiModelProperty(value = "潜在客户归属")
        private String lead_owner;
        @JsonProperty("SALUTATION")
        @ApiModelProperty(value = "称谓")
        private String salutation;
        @JsonProperty("LAST_NAME")
        @ApiModelProperty(value = "姓氏")
        private String last_name;
        @JsonProperty("MIDDLE_NAME")
        @ApiModelProperty(value = "中间名")
        private String middle_name;
        @JsonProperty("FIRST_NAME")
        @ApiModelProperty(value = "名")
        private String first_name;
        @JsonProperty("SUFFIX")
        @ApiModelProperty(value = "别名")
        private String suffix;
        @JsonProperty("WEBSITE")
        @ApiModelProperty(value = "网址")
        private String website;
        @JsonProperty("TITLE")
        @ApiModelProperty(value = "职位")
        private String title;
        @JsonProperty("COMPANY")
        @ApiModelProperty(value = "所属企业")
        private String company;
        @JsonProperty("EMAIL")
        @ApiModelProperty(value = "邮件地址")
        private String email;
        @JsonProperty("INDUSTRY")
        @ApiModelProperty(value = "所属行业")
        private String industry;
        @JsonProperty("PHONE")
        @ApiModelProperty(value = "电话号码")
        private String phone;
        @JsonProperty("MOBILE")
        @ApiModelProperty(value = "手机号码")
        private String mobile;

        @JsonProperty("EMPLOYEES")
        @ApiModelProperty(value = "员工数量")
        private String employees;
        @JsonProperty("LEAD_SOURCE")
        @ApiModelProperty(value = "潜在客户来源")
        private String lead_source;
        @JsonProperty("RATING")
        @ApiModelProperty(value = "分级")
        private String rating;
        @JsonProperty("ADDRESS_ID")
        @ApiModelProperty(value = "潜在客户地址")
        private String address_id;
        @JsonProperty("CREATE_LOGIN")
        @ApiModelProperty(value = "创建人工号")
        private String create_login;
        @JsonProperty("CREATE_NAME")
        @ApiModelProperty(value = "创建人名称")
        private String create_name;
        @JsonProperty("CREATE_TIME")
        @ApiModelProperty(value = "创建时间")
        private Date create_time;
        @JsonProperty("OPERATE_LOGIN")
        @ApiModelProperty(value = "操作人工号")
        private String operate_login;
        @JsonProperty("OPERATE_NAME")
        @ApiModelProperty(value = "操作人名称")
        private String operate_name;
        @JsonProperty("OPERATE_TIME")
        @ApiModelProperty(value = "操作时间")
        private Date operate_time;
		
    	//人员信息的get set方法
    //输出地址信息
    @JsonProperty("ENT_ID")
    @ApiModelProperty(value = "潜在客户企业标识")
    private String ent_id;
    @JsonProperty("COUNTRY")
    @ApiModelProperty(value = "潜在客户国家")
    private String country;
    @JsonProperty("PROVINCE")
    @ApiModelProperty(value = "潜在客户省份")
    private String province;
    @JsonProperty("CITY")
    @ApiModelProperty(value = "潜在客户城市")
    private String city;
    @JsonProperty("DISTRICT")
    @ApiModelProperty(value = "潜在客户区县")
    private String district;
    @JsonProperty("STREET")
    @ApiModelProperty(value = "潜在客户街道")
    private String street;
    @JsonProperty("POSTCODE")
    @ApiModelProperty(value = "潜在客户邮政编码")
    private String postcode;

   //地址信息的get set方法
	
    //建造者方式进行后端set get的改造
    public static ProspectiveCustOut.Builder builder() {
        return new ProspectiveCustOut.Builder();
    }

    public static class Builder {
        private ProspectiveCustOut obj;

        public Builder() {
            this.obj = new ProspectiveCustOut();
        }

        public ProspectiveCustOut build() {
            return this.obj;
        }

        public Builder lead_id(String lead_id) {
            obj.setLead_id(lead_id);
            return this;
        }

        public Builder status(String status) {
            obj.setStatus(status);
            return this;
        }

        public Builder end_id(String end_id) {
            obj.setEnd_id(end_id);
            return this;
        }

        public Builder lead_owner(String lead_owner) {
            obj.setLead_source(lead_owner);
            return this;
        }

        public Builder salutation(String salutation) {
            obj.setSalutation(salutation);
            return this;
        }

        public Builder last_name(String last_name) {
            obj.setLast_name(last_name);
            return this;
        }

        public Builder middle_name(String middle_name) {
            obj.setMiddle_name(middle_name);
            return this;
        }

        public Builder first_name(String first_name) {
            obj.setFirst_name(first_name);
            return this;
        }

        public Builder suffix(String suffix) {
            obj.setSuffix(suffix);
            return this;
        }

        public Builder website(String website) {
            obj.setWebsite(website);
            return this;
        }

        public Builder title(String title) {
            obj.setTitle(title);
            return this;
        }

        public Builder company(String company) {
            obj.setCompany(company);
            return this;
        }

        public Builder email(String email) {
            obj.setEmail(email);
            return this;
        }

        public Builder industry(String industry) {
            obj.setIndustry(industry);
            return this;
        }

        public Builder phone(String phone) {
            obj.setPhone(phone);
            return this;
        }

        public Builder mobile(String mobile) {
            obj.setMobile(mobile);
            return this;
        }

        public Builder employees(String employees) {
            obj.setEmployees(employees);
            return this;
        }

        public Builder lead_source(String lead_source) {
            obj.setLead_source(lead_source);
            return this;
        }

        public Builder rating(String rating) {
            obj.setRating(rating);
            return this;
        }

        public Builder address_id(String address_id) {
            obj.setAddress_id(address_id);
            return this;
        }

        public Builder create_login(String create_login) {
            obj.setCreate_login(create_login);
            return this;
        }

        public Builder create_name(String create_name) {
            obj.setCreate_name(create_name);
            return this;
        }

        public Builder create_time(String create_time) {
            obj.setCreate_name(create_time);
            return this;
        }

        public Builder operate_login(String operate_login) {
            obj.setOperate_login(operate_login);
            return this;
        }

        public Builder operate_name(String operate_name) {
            obj.setOperate_name(operate_name);
            return this;
        }

        public Builder operate_time(Date operate_time) {
            obj.setOperate_time(operate_time);
            return this;
        }
        public Builder ent_id(String ent_id) {
            obj.setEnt_id(ent_id);
            return this;
        }
        public Builder country(String country) {
            obj.setCountry(country);
            return this;
        }
        public Builder province(String province) {
            obj.setProvince(province);
            return this;
        }
        public Builder city(String city) {
            obj.setCity(city);
            return this;
        }
        public Builder district(String district) {
            obj.setDistrict(district);
            return this;
        }
        public Builder street(String street) {
            obj.setStreet(street);
            return this;
        }
        public Builder postcode(String postcode) {
            obj.setPostcode(postcode);
            return this;
        }
    }
}

```

mapper文件

```java
package com.sitech.salecenter.salecenterservice.mapper;

import com.sitech.salecenter.salecenterservice.dao.domain.SeLeadcustInfo;
import com.sitech.salecenter.salecenterservice.dao.domain.SeLeadcustInfoExample;
import java.util.List;
import org.apache.ibatis.annotations.Param;

public interface SeLeadcustInfoMapper {
    long countByExample(SeLeadcustInfoExample example);

    int deleteByExample(SeLeadcustInfoExample example);

    int deleteByPrimaryKey(String leadId);

    int insert(SeLeadcustInfo record);

    int insertSelective(@Param("record") SeLeadcustInfo record, @Param("selective") SeLeadcustInfo.Column ... selective);

    SeLeadcustInfo selectOneByExample(SeLeadcustInfoExample example);

    List<SeLeadcustInfo> selectByExample(SeLeadcustInfoExample example);

    SeLeadcustInfo selectByPrimaryKey(String leadId);

    int updateByExampleSelective(@Param("record") SeLeadcustInfo record, @Param("example") SeLeadcustInfoExample example, @Param("selective") SeLeadcustInfo.Column ... selective);

    int updateByExample(@Param("record") SeLeadcustInfo record, @Param("example") SeLeadcustInfoExample example);

    int updateByPrimaryKeySelective(@Param("record") SeLeadcustInfo record, @Param("selective") SeLeadcustInfo.Column ... selective);

    int updateByPrimaryKey(SeLeadcustInfo record);

    int batchInsert(@Param("list") List<SeLeadcustInfo> list);

    int batchInsertSelective(@Param("list") List<SeLeadcustInfo> list, @Param("selective") SeLeadcustInfo.Column ... selective);
}
```

controller

```java

package com.sitech.salecenter.salecenterservice.service.leadcustinfo.inter;

import com.sitech.ijcf.message6.dt.in.InDTO;
import com.sitech.ijcf.message6.dt.out.OutDTO;
import com.sitech.salecenter.salecenterservice.dto.in.*;
import com.sitech.salecenter.salecenterservice.dto.out.ProspectiveCustOut;
import io.swagger.annotations.ApiParam;
import org.springframework.web.bind.annotation.RequestBody;

import java.util.List;
import java.util.Map;

public interface IProspectiveCustService {
     public OutDTO<List<ProspectiveCustOut>> findCust(InDTO<ProspectiveCustIn> prospectiveCustInInDTO);
     
    public OutDTO<String> createCust(InDTO<ProspectiveCustInCreate> prospectiveCustInCreateInDTO);

     public OutDTO<Integer> deleteByCustId(InDTO<ProspectiveCustInDel> prospectiveCustInDelInDTO);

     public OutDTO<ProspectiveCustOut> updateCustInfo(InDTO<ProspectiveCustInUpdate> prospectiveCustInUpdateInDTO);
     public List<Map<String, Object>> findCustAll();
    
}

```

```java
package com.sitech.salecenter.salecenterservice.service.leadcustinfo;

import com.sitech.ijcf.message6.dt.in.InDTO;
import com.sitech.ijcf.message6.dt.out.OutDTO;
import com.sitech.salecenter.salecentercommon.util.pkey.Sequence;
import com.sitech.salecenter.salecentercommon.util.pkey.SequenceEnum;
import com.sitech.salecenter.salecenterservice.dao.domain.SeAddressInfo;
import com.sitech.salecenter.salecenterservice.dao.domain.SeAddressInfoExample;
import com.sitech.salecenter.salecenterservice.dao.domain.SeLeadcustInfo;
import com.sitech.salecenter.salecenterservice.dao.domain.SeLeadcustInfoExample;
import com.sitech.salecenter.salecenterservice.dto.in.*;
import com.sitech.salecenter.salecenterservice.dto.out.ProspectiveCustOut;
import com.sitech.salecenter.salecenterservice.mapper.SeAddressInfoMapper;
import com.sitech.salecenter.salecenterservice.mapper.SeLeadcustInfoMapper;
import com.sitech.salecenter.salecenterservice.service.leadcustinfo.inter.IProspectiveCustService;
import com.sitech.salecenter.salecenterservice.util.DatesUtil;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import static com.sitech.ijcf.boot.core.util.JXml.log;
import javax.annotation.Resource;
import java.util.*;

/**
 * @author yuzp
 * @date 2019/9/4/14:02
 */

@Api(tags = "潜在客户查询系统", description = "潜在客户查询系统", position = 300)
@Service("leadCustServiceSvc")
public class ProspectiveCustServiceImpl implements IProspectiveCustService {
    @Resource
    SeLeadcustInfoMapper seLeadcustInfoMapper;
    @Resource
    Sequence sequence;
    @Resource
    SeAddressInfoMapper seAddressInfoMapper;
    @Override
    @RequestMapping(value = "/leadcust", method = RequestMethod.POST)
    @ApiOperation(value = "查询潜在客户-根据不同前端入参", position = 1)
    public OutDTO<List<ProspectiveCustOut>> findCust(@RequestBody @ApiParam("请求入参") InDTO<ProspectiveCustIn>
                                                             prospectiveCustInInDTO) {
        ProspectiveCustIn custIn = prospectiveCustInInDTO.getBody();
        String flag=custIn.getFlag();
        List<ProspectiveCustOut> resultList = new ArrayList<ProspectiveCustOut>();
        if("firstpage".equals(flag)){
            resultList=findCustById(custIn.getLead_id());
        }else if("fuzzyquery".equals(flag)){
            resultList= findCustByLastName(custIn.getLast_name());
        }
        else if ("date".equals(flag)){
            resultList=findCustByCreateTime();
        }
        else if ("findall".equals(flag)){
            resultList=findAllCust();
        }
        OutDTO<List<ProspectiveCustOut>> listOutDTO = new OutDTO<>();
        listOutDTO.setBodyOutData(resultList);
        return listOutDTO;
    }

    public List<ProspectiveCustOut> findCustById(String id) {
        //声明入参对象，并得到body
        SeLeadcustInfoExample seLeadcustInfoExample = new SeLeadcustInfoExample();
        SeLeadcustInfoExample.Criteria criteria = seLeadcustInfoExample.createCriteria();
        criteria.andLeadIdEqualTo(id);
        List<SeLeadcustInfo> leadcustInfos = seLeadcustInfoMapper.selectByExample(seLeadcustInfoExample);

        SeLeadcustInfo seLeadcustInfo1 = seLeadcustInfoMapper.selectByPrimaryKey(id);
        SeAddressInfo seAddressInfo1 = seAddressInfoMapper.selectByPrimaryKey(seLeadcustInfo1.getAddressId());
        String address_id=seAddressInfo1.getAddressId();

        SeAddressInfoExample seAddressInfoExample = new SeAddressInfoExample();
        SeAddressInfoExample.Criteria criteria1 = seAddressInfoExample.createCriteria();
        criteria1.andAddressIdEqualTo(address_id);
        List<SeAddressInfo> seAddressInfos = seAddressInfoMapper.selectByExample(seAddressInfoExample);

        //声明List
        List<ProspectiveCustOut> prospectiveCustOuts = new ArrayList<>();
        //遍历出参，并添加到声明的list列表中
        for (SeLeadcustInfo seLeadcustInfo : leadcustInfos) {
            ProspectiveCustOut out = ProspectiveCustOut.builder()
                    .email(seLeadcustInfo.getEmail())
                    .company(seLeadcustInfo.getCompany())
                    .phone(seLeadcustInfo.getPhone())
                    .title(seLeadcustInfo.getTitle())
                    .mobile(seLeadcustInfo.getMobile())
                    .status((seLeadcustInfo.getStatus()))
                    .build();
            prospectiveCustOuts.add(out);
        }
        for (SeAddressInfo seAddressInfo:seAddressInfos){
            ProspectiveCustOut out = ProspectiveCustOut.builder()
                    .ent_id(seAddressInfo.getEntId())
                    .country(seAddressInfo.getCountry())
                    .city(seAddressInfo.getCity())
                    .district(seAddressInfo.getDistrict())
                    .street(seAddressInfo.getStreet())
                    .postcode(seAddressInfo.getPostcode())
                    .build();
            prospectiveCustOuts.add(out);
        }
        return prospectiveCustOuts;
    }
    public List<ProspectiveCustOut> findCustByLastName(String name) {
        long time = System.currentTimeMillis();
        SeLeadcustInfoExample seLeadcustInfoExampleLike = new SeLeadcustInfoExample();
        SeLeadcustInfoExample.Criteria criteriaLike = seLeadcustInfoExampleLike.createCriteria();
        //注意模糊查询中的参数写法
        criteriaLike.andLastNameLike("%" + name + "%");
        List<SeLeadcustInfo> leadcustInfosLike = seLeadcustInfoMapper.selectByExample(seLeadcustInfoExampleLike);

        //遍历出参，并添加到声明的list列表中
        List<ProspectiveCustOut> prospectiveCustOutsLike = new ArrayList<>();
        for (int i = 0; i < leadcustInfosLike.size(); i++) {
            String address_id=leadcustInfosLike.get(i).getAddressId();
            SeAddressInfo seAddressInfo = seAddressInfoMapper.selectByPrimaryKey(address_id);
            ProspectiveCustOut out = ProspectiveCustOut.builder()
                    .last_name(leadcustInfosLike.get(i).getLastName())
                    .title(leadcustInfosLike.get(i).getTitle())
                    .company(leadcustInfosLike.get(i).getCompany())
                    .phone(leadcustInfosLike.get(i).getPhone())
                    .mobile(leadcustInfosLike.get(i).getMobile())
                    .email(leadcustInfosLike.get(i).getEmail())
                    .status(leadcustInfosLike.get(i).getStatus())
                    .suffix(leadcustInfosLike.get(i).getSuffix())
                    .ent_id(seAddressInfo.getEntId())
                    .country(seAddressInfo.getCountry())
                    .city(seAddressInfo.getCity())
                    .district(seAddressInfo.getDistrict())
                    .street(seAddressInfo.getStreet())
                    .postcode(seAddressInfo.getPostcode())
                    .build();
            prospectiveCustOutsLike.add(out);
        }
        log.info("执行时间：" + (System.currentTimeMillis() - time));
        return prospectiveCustOutsLike;
    }

    public List<ProspectiveCustOut> findCustByCreateTime() {
        Date date = DatesUtil.getDayBegin();
        Date date1 = DatesUtil.getDayEnd();
        //划分时间线，使用Data公共类来确定当天时间的截止与确定
        SeLeadcustInfoExample seLeadcustInfoExampleByTime = new SeLeadcustInfoExample();
        SeLeadcustInfoExample.Criteria criteriaByTime = seLeadcustInfoExampleByTime.createCriteria();
        criteriaByTime.andCreateTimeBetween(date, date1);
        List<SeLeadcustInfo> leadcustInfosByTime = seLeadcustInfoMapper.selectByExample(seLeadcustInfoExampleByTime);

        List<ProspectiveCustOut> prospectiveCustOutsByTime = new ArrayList<>();
        for(int i =0;i<leadcustInfosByTime.size();i++){
            String address_id = leadcustInfosByTime.get(i).getAddressId();
            SeAddressInfo seAddressInfo = seAddressInfoMapper.selectByPrimaryKey(address_id);
            ProspectiveCustOut out = ProspectiveCustOut.builder()
                    .last_name(leadcustInfosByTime.get(i).getLastName())
                    .title(leadcustInfosByTime.get(i).getTitle())
                    .company(leadcustInfosByTime.get(i).getCompany())
                    .phone(leadcustInfosByTime.get(i).getPhone())
                    .mobile(leadcustInfosByTime.get(i).getMobile())
                    .email(leadcustInfosByTime.get(i).getEmail())
                    .status(leadcustInfosByTime.get(i).getStatus())
                    .suffix(leadcustInfosByTime.get(i).getSuffix())

                    .ent_id(seAddressInfo.getEntId())
                    .country(seAddressInfo.getCountry())
                    .city(seAddressInfo.getCity())
                    .district(seAddressInfo.getDistrict())
                    .street(seAddressInfo.getStreet())
                    .postcode(seAddressInfo.getPostcode())
                    .build();
            prospectiveCustOutsByTime.add(out);
        }
        return prospectiveCustOutsByTime;
    }

    public List<ProspectiveCustOut> findAllCust(){
        SeLeadcustInfoExample seLeadcustInfoExample = new SeLeadcustInfoExample();
        SeLeadcustInfoExample.Criteria criteria = seLeadcustInfoExample.createCriteria();
        criteria.andLeadIdIsNotNull();
        List<SeLeadcustInfo> leadcustInfos = seLeadcustInfoMapper.selectByExample(seLeadcustInfoExample);

        List<ProspectiveCustOut> list = new ArrayList<>();
        for(int i=0;i<leadcustInfos.size();i++)
        {
            String address_id = leadcustInfos.get(i).getAddressId();
            SeAddressInfo seAddressInfo = seAddressInfoMapper.selectByPrimaryKey(address_id);
            ProspectiveCustOut prospectiveCustOut= ProspectiveCustOut.builder()
                    .last_name(leadcustInfos.get(i).getLastName())
                .title(leadcustInfos.get(i).getTitle())
                .company(leadcustInfos.get(i).getCompany())
                .phone(leadcustInfos.get(i).getPhone())
                .mobile(leadcustInfos.get(i).getMobile())
                .email(leadcustInfos.get(i).getEmail())
                .status(leadcustInfos.get(i).getStatus())
                .suffix(leadcustInfos.get(i).getSuffix())
                .ent_id(seAddressInfo.getEntId())
                .country(seAddressInfo.getCountry())
                .city(seAddressInfo.getCity())
                .district(seAddressInfo.getDistrict())
                .street(seAddressInfo.getStreet())
                .postcode(seAddressInfo.getPostcode())
                .build();
            list.add(prospectiveCustOut);
        }
        return list;
    }

    @Autowired
    private JdbcTemplate jdbcTemplate;
    @Override
    @RequestMapping(value = "/leadcustall", method = RequestMethod.POST)
    @ApiOperation(value = "查询潜在客户-全查询", position = 1)
    public List<Map<String, Object>> findCustAll() {
        String sql = "select * from se_leadcust_info";
        List<Map<String, Object>> list = jdbcTemplate.queryForList(sql);
        for (Map<String, Object> map : list) {
            //该方法返回值就是这个map中各个键值对映射关系的集合
            Set<Map.Entry<String, Object>> entries = map.entrySet();
            if (entries != null) {
                Iterator<Map.Entry<String, Object>> iterator = entries.iterator();
                //对iterator进行while循环，并且输出相应的key value
                while (iterator.hasNext()) {
                    Map.Entry<String, Object> entry = iterator.next();
                    Object key = entry.getKey();
                    Object value = entry.getValue();
                    System.out.println(key + ":" + value);
                }
            }
        }
        return list;
    }

    @Override
    @RequestMapping(value = "/createcust", method = RequestMethod.POST)
    @ApiOperation(value = "创建客户", position = 1)
    public OutDTO<String> createCust(@RequestBody @ApiParam("创建客户") InDTO<ProspectiveCustInCreate>
                                             prospectiveCustInCreateInDTO) {
        ProspectiveCustInCreate prospectiveCustInCreateCust = prospectiveCustInCreateInDTO.getBody();
        //随机数
        String contactId = sequence.getSequence(SequenceEnum.PROJECT_ID);
        //时间字段
        Date date = new Date();
        java.sql.Date date1 = new java.sql.Date(date.getTime());
        String addressId = sequence.getSequence(SequenceEnum.PROJECT_ID);
        //页面上显示的字段
        SeLeadcustInfo seLeadcustInfo = SeLeadcustInfo.builder()
                .status(prospectiveCustInCreateCust.getStatus())
                .salutation(prospectiveCustInCreateCust.getSalutation())
                .website(prospectiveCustInCreateCust.getWebsite())
                .lastName(prospectiveCustInCreateCust.getLast_name())
                .middleName(prospectiveCustInCreateCust.getMiddle_name())
                .firstName(prospectiveCustInCreateCust.getFirst_name())
                .title(prospectiveCustInCreateCust.getTitle())
                .company(prospectiveCustInCreateCust.getCompany())
                .email(prospectiveCustInCreateCust.getEmail())
                .industry(prospectiveCustInCreateCust.getIndustry())
                .phone(prospectiveCustInCreateCust.getPhone())
                .employees(prospectiveCustInCreateCust.getEmployees())
                .mobile(prospectiveCustInCreateCust.getMobile())
                .rating(prospectiveCustInCreateCust.getRating())
                .leadSource(prospectiveCustInCreateCust.getLead_source())
                //页面上没有显示的字段
                .entId(prospectiveCustInCreateCust.getEnd_id())
                .leadOwner(prospectiveCustInCreateCust.getLead_owner())
                .suffix(prospectiveCustInCreateCust.getSuffix())
                .addressId(addressId)
                .createLogin(prospectiveCustInCreateCust.getCreate_login())
                .createName(prospectiveCustInCreateCust.getCreate_name())
                .operateLogin(prospectiveCustInCreateCust.getOperate_login())
                .createName(prospectiveCustInCreateCust.getCreate_name())
                .leadId(contactId)
                .createTime(date1)
                .build();
        SeAddressInfo seAddressInfo = SeAddressInfo.builder()
                .addressId(addressId)
                .entId(prospectiveCustInCreateCust.getEnt_id())
                .country(prospectiveCustInCreateCust.getCountry())
                .province(prospectiveCustInCreateCust.getProvince())
                .city(prospectiveCustInCreateCust.getCity())
                .district(prospectiveCustInCreateCust.getDistrict())
                .street(prospectiveCustInCreateCust.getStreet())
                .postcode(prospectiveCustInCreateCust.getPostcode())
                .build();
        int result = seLeadcustInfoMapper.insert(seLeadcustInfo);
        int result1 = seAddressInfoMapper.insert(seAddressInfo);
        OutDTO<String> outDTO = new OutDTO<>();
        if (result  > 0) {
            outDTO.setBodyOutData("ok");
        }
        outDTO.setBodyOutData("default");
        return outDTO;
    }
    @Override
    @ApiOperation(value = "删除潜在客户")
    @RequestMapping(value = "/deleteLeadCust", method = RequestMethod.POST)
    public OutDTO<Integer> deleteByCustId(@RequestBody @ApiParam(value = "根据id删除潜在客户") InDTO<ProspectiveCustInDel>
                                                  prospectiveCustInDelInDTO) {
        ProspectiveCustInDel body = prospectiveCustInDelInDTO.getBody();
        //根据主键删除潜在客户
        String id = body.getLead_id();
        //根据主键获得地址id，然后删除地址表中的数据
        SeLeadcustInfo seLeadcustInfo = seLeadcustInfoMapper.selectByPrimaryKey(id);
        int i = seAddressInfoMapper.deleteByPrimaryKey(seLeadcustInfo.getAddressId());
        int del = seLeadcustInfoMapper.deleteByPrimaryKey(id);
        OutDTO<Integer> outDTO = new OutDTO<>();
        outDTO.setBodyOutData(del);
        outDTO.setBodyOutData(i);
        return outDTO;
    }

    @Override
    @ApiOperation(value = "更新潜在客户的信息")
    @RequestMapping(value = "/updateCustInfo", method = RequestMethod.POST)
    public OutDTO<ProspectiveCustOut> updateCustInfo(@RequestBody @ApiParam("更新潜在客户信息") InDTO<ProspectiveCustInUpdate>
                                                             prospectiveCustInUpdateInDTO) {
        ProspectiveCustInUpdate prospectiveCustInUpdate = prospectiveCustInUpdateInDTO.getBody();
        //主键
        String lead_id = prospectiveCustInUpdate.getLead_id();

        //要修改的字段
        String status = prospectiveCustInUpdate.getStatus();
        String website = prospectiveCustInUpdate.getWebsite();
        String salutation = prospectiveCustInUpdate.getSalutation();
        String last_name = prospectiveCustInUpdate.getLast_name();
        String middle_name = prospectiveCustInUpdate.getMiddle_name();
        String first_name = prospectiveCustInUpdate.getFirst_name();
        String title = prospectiveCustInUpdate.getTitle();
        String company = prospectiveCustInUpdate.getCompany();
        String email = prospectiveCustInUpdate.getEmail();
        String industry = prospectiveCustInUpdate.getIndustry();
        String phone = prospectiveCustInUpdate.getPhone();
        String employees = prospectiveCustInUpdate.getEmployees();
        String mobile = prospectiveCustInUpdate.getMobile();
        String lead_source = prospectiveCustInUpdate.getLead_source();
        String rating = prospectiveCustInUpdate.getRating();
        //根据lead id修改
        String update = prospectiveCustInUpdate.getUpdate();
        SeLeadcustInfoExample seLeadcustInfoExample = new SeLeadcustInfoExample();
        SeLeadcustInfoExample.Criteria criteria = seLeadcustInfoExample.createCriteria();
        criteria.andLeadIdEqualTo(lead_id);
        //数字为自定义，意义不明确，要与前端沟通之后确认
        Map<String, String> map = new HashMap<>();
        map.put("1", status);
        map.put("2", website);
        map.put("3", salutation);
        map.put("4", last_name);
        map.put("5", middle_name);
        map.put("6", first_name);
        map.put("7", title);
        map.put("8", company);
        map.put("9", email);
        map.put("10", industry);
        map.put("11", phone);
        map.put("12", employees);
        map.put("13", mobile);
        map.put("14", lead_source);
        map.put("15", rating);

        SeLeadcustInfo seLeadcustInfo = SeLeadcustInfo.builder()
                .status(status)
                .website(website)
                .salutation(salutation)
                .lastName(last_name)
                .middleName(middle_name)
                .firstName(first_name)
                .title(title)
                .company(company)
                .email(email)
                .industry(industry)
                .phone(phone)
                .employees(employees)
                .mobile(mobile)
                .leadSource(lead_source)
                .rating(rating)
                .build();
        seLeadcustInfoMapper.updateByExampleSelective(seLeadcustInfo, seLeadcustInfoExample);
        return null;
    }


}


```

