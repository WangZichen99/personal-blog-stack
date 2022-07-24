---
title: SpringBoot笔记
sult: SpringBootNote
author: wzc
date: 2022-07-09T11:18:24+08:00
categories: 
    - Study Note
tags: 
    - Spring
    - Spring MVC
    - Spring Boot
image: img/2022/07/nature_park.jpg
draft: false
---

# SpringBoot

## 1、原理初探

**pom.xml**
- spring-boot-dependencies：核心依赖，在父工程中
- 在引入springboot依赖时不需要指定版本就是因为有这个版本库

**启动器**
- ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    ```
- 启动器就是springboot的启动场景，会自动导入-web环境的所有依赖
- springboot将所有的功能场景，都变成了一个个启动器
- 以后需要什么功能就直接添加启动器就可以了

**主程序**
@SpringBootApplication：标注这个类是一个springboot的应用

![img.png](../../img/2022/07/springboot_note/img.png)

@SpringBootConfiguration：@SpringBootConfiguration是springboot的配置，其底层还是@Configuration

![img_2.png](../../img/2022/07/springboot_note/img_2.png)

@EnableAutoConfiguration：自动配置注解（重要）

![img_1.png](../../img/2022/07/springboot_note/img_1.png)

@AutoConfigurationPackage：自动配置包

@Import({Registrar.class})：导入自动配置包的注册类

@Import({AutoConfigurationImportSelector.class})：导入了AutoConfigurationImportSelector选择器

![img_3.png](../../img/2022/07/springboot_note/img_3.png)

```
//获取自动配置的实体的方法
getAutoConfigurationEntry(AutoConfigurationMetadata, AnnotationMetadata)
//方法中获取候选配置项
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
```

![img_4.png](../../img/2022/07/springboot_note/img_4.png)

```
//获取所有EnableAutoConfiguration注解的配置项返回
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, beanClassLoader);
```

![img_5.png](../../img/2022/07/springboot_note/img_5.png)

```
//从META-INF/spring.factories"中获取项目资源和系统资源（遍历文件后封装成Properties）
loadSpringFactories(ClassLoader)
```

![img_6.png](../../img/2022/07/springboot_note/img_6.png)

spring-boot-autoconfigure.jar/META-INF/spring.factories文件中包含了所有springboot的自动配置

![img_7.png](../../img/2022/07/springboot_note/img_7.png)
![img_8.png](../../img/2022/07/springboot_note/img_8.png)

spring.factories中的AutoConfiguration类都是容器中的一个组件，最后都加入到容器中
当满足了@ConditionalOnClass注解中的条件之后（就导入了对应的starter），自动配置才会生效
自动配置生效后，各个配置类就会向容器中添加各种组件，这些组件的属性值就是从对应的properties文件中
获取的，并且这些属性与yaml配置文件是绑定的。

![img_9.png](../../img/2022/07/springboot_note/img_9.png)

@ConfigurationProperties绑定配置文件中的配置

![img_10.png](../../img/2022/07/springboot_note/img_10.png)

**自动配置就是springboot自动将spring.factories中的组件注入到容器中供用户直接使用**

1. 判断应用的类型是不是Web项目
2. 查找并加载所有可用的初始化器，设置到initializers属性中
3. 找出所有的应用程序监听器，设置到listener中
4. 设置main方法的定义类，加载运行的主类

可以通过在配置文件中配置debug=true来查看哪些配置类生效了，哪些没有生效

## 2、YMAL配置文件
```yaml
#键值对
name: wzc

#对象
student:
  name: wzc
  age: 22
  
stu: {name: wzc, age: 22} #行内写法

#数组
subject:
  - chinese
  - math
  - english
    
sub: [chinese, math, english] #行内写法
```

## 2.1、给属性赋值的几种方式

1. 使用ymal
```yaml
person:
  name: wzc
  age: 22
  happy: true
  birthday: 1999/10/12
  map:
    hobby: play
  list:
    - a
    - b
    - c
  dog:
    name: niuniu
    age: 3
```
```java
@Component
@Data
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birthday;
    private Map<String, Object> map;
    private List<Object> list;
    private Dog dog;
}
```
@ConfigurationProperties注解会从配置文件中寻找和prefix同名的对象注入到容器中

2. 使用properties
   person.properties
```properties
name=wzc
age=22
```

```java
@Component
@Data
@PropertySource("classpath:person.properties")
public class Person {
    @Value("${name}")
    private String name;
    @Value("${age}")
    private Integer age;
    private Boolean happy;
    private Date birthday;
    private Map<String, Object> map;
    private List<Object> list;
    private Dog dog;
}
```
使用@PropertySource选择properties文件，然后使用spel表达式对属性进行赋值

**松散绑定**

在yml中的-后的字母默认是大写的，last-name对应着实体类中的lastName

**JSR303数据校验**

```
@Null
@NotNull
...
```

## 2.2、多环境配置
```yaml
spring:
  profiles:
    active: dev
---
spring:
  profiles: dev
server:
  port: 8080
---
spring:
  profiles: test
server:
    port: 8081
---
spring:
  profiles: pro
server:
    port: 8082
```

# 3、web开发
## 3.1、静态资源
看源码：WebMvcAutoConfiguration.class
```java
public class WebMvcAutoConfiguration {
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        /**
         * 第三种方式：自己配置spring.mvc.static-path-pattern
         */
        if (!this.resourceProperties.isAddMappings()) {
            logger.debug("Default resource handling disabled");
        } else {
            Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
            CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
            /**
             * 第一种方法：使用webjars引入静态资源，webjars会以maven的形式导入静态资源，
             * springboot会到maven包中的/META-INF/resources/webjars/目录下寻找静态资源
              */
            if (!registry.hasMappingForPattern("/webjars/**")) {
                this.customizeResourceHandlerRegistration(registry
                        .addResourceHandler(new String[]{"/webjars/**"})
                        .addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"})
                        .setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
            }

            /**
             * 第二种方法：springboot会先从mvcProperties中获取staticPathPattern，默认值为/**
             * 然后会设置resourceLocations，默认值为"classpath:/META-INF/resources/", 
             * "classpath:/resources/", "classpath:/static/", "classpath:/public/"
             */
            String staticPathPattern = this.mvcProperties.getStaticPathPattern();
            if (!registry.hasMappingForPattern(staticPathPattern)) {
                this.customizeResourceHandlerRegistration(registry
                        .addResourceHandler(new String[]{staticPathPattern})
                        .addResourceLocations(WebMvcAutoConfiguration
                                .getResourceLocations(this.resourceProperties.getStaticLocations()))
                        .setCachePeriod(this.getSeconds(cachePeriod))
                        .setCacheControl(cacheControl));
            }
        }
    }
}

```

## 3.2 首页如何定制
看源码：WebMvcAutoConfiguration.class
![img_11.png](../../img/2022/07/springboot_note/img_11.png)

## 3.3、Thymeleaf模板引擎
1. 导入
```xml
<dependencies>
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
    </dependency>

    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-java8time</artifactId>
    </dependency>
</dependencies>
```

2. 将静态文件放在templates目录下

![img_12.png](../../img/2022/07/springboot_note/img_12.png)

3. 编写视图解析器
```java
@Controller
public class IndexController {
    @GetMapping("/index")
    public String index() {
        return "index";
    }
}
```

```html
<!-- 所有的html元素都可以被thymeleaf接管，th:元素名 -->
<p th:text="${msg}"></p>
<!-- 带转义的text -->
<p th:utext="${msg}"></p>
```

## 3.4、SpringMVC配置

1. 自定义视图解析器
```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    /**
     * 如果想自定义mvc的功能，实现mvc的组件，然后交给springboot会自动装配
     * @return
     */
    @Bean
    public ViewResolver getViewResolver() {
        return new ViewResolver();
    }

    /**
     * 自动以视图解析器，实现ViewResolver接口
     */
    public static class ViewResolver implements org.springframework.web.servlet.ViewResolver {
        @Override
        public View resolveViewName(String s, Locale locale) throws Exception {
            return null;
        }
    }
}
```

2. 日期转换

默认yyyy/MM/dd

![img_13.png](../../img/2022/07/springboot_note/img_13.png)

3. 视图跳转

![img_14.png](../../img/2022/07/springboot_note/img_14.png)

**如果需要扩展SpringMvc，官方建议在类上添加@Configuration注解，并实现WebMVCConfigurer接口，然后重写其中的方法**

## 3.5、员工管理系统

### 3.5.1、首页实现

使用视图跳转让thymeleaf接管
```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    /**
     * 视图跳转
     */
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/index").setViewName("index");
        registry.addViewController("/home").setViewName("index");
    }
}
```

在页面中url地址使用@{}

![img_15.png](../../img/2022/07/springboot_note/img_15.png)

### 3.5.2、国际化
1. 建立三个properties文件分别配置页面上需要国际化的不同语言

![img_16.png](../../img/2022/07/springboot_note/img_16.png)

2. 修改html页面中对应的

![img_17.png](../../img/2022/07/springboot_note/img_17.png)

3. 中英文切换按钮，需要自定义LocaleResolver组件并注入到spring容器中

![img_18.png](../../img/2022/07/springboot_note/img_18.png)

```java
public class LocalResolver implements LocaleResolver {
    /**
     * 解析请求
     * @param request
     * @return
     */
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String language = request.getParameter("lang");
        if (!StringUtils.isEmpty(language)) {
            String[] split = language.split("_");
            return new Locale(split[0], split[1]);
        }
        return Locale.getDefault();
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}
```

![img_19.png](../../img/2022/07/springboot_note/img_19.png)

### 3.5.3、登录和登录拦截器

![img_20.png](../../img/2022/07/springboot_note/img_20.png)

登录和退出接口
```java
@Controller
public class LoginController {
    @PostMapping("/login")
    public String login(@RequestParam("username") String username, @RequestParam("password") String password, Model model, HttpSession httpSession) {
        if (StringUtils.isEmpty(username) || StringUtils.isEmpty(password)) {
            model.addAttribute("msg", "用户名或密码错误！");
            return "index";
        }
        httpSession.setAttribute("loginUser", username);
        return "redirect:/dashboard";
    }

    @GetMapping("/signOut")
    public String signOut(HttpSession httpSession) {
        httpSession.removeAttribute("loginUser");
        return "index";
    }
}
```

用户可以直接访问dashboard页面，所以需要配置拦截器
```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (ObjectUtils.isEmpty(request.getSession().getAttribute("loginUser"))) {
            request.setAttribute("msg", "没有权限，请先登录！");
            request.getRequestDispatcher("/index").forward(request, response);
            return false;
        }
        return true;
    }
}
```

MvcConfig中配置拦截器

![img_21.png](../../img/2022/07/springboot_note/img_21.png)

### 3.5.4、查询信息和CRUD
实际开发中通常将通用的前端代码提取出来。使用th:fragment

![img_22.png](../../img/2022/07/springboot_note/img_22.png)

在页面中替换代码片段th:replace

![img_23.png](../../img/2022/07/springboot_note/img_23.png)

循环遍历数据th:each
```html
<table class="table table-striped table-sm">
    <thead>
        <tr>
            <th>id</th>
            <th>lastName</th>
            <th>email</th>
            <th>gender</th>
            <th>department</th>
            <th>birthday</th>
            <th>operation</th>
        </tr>
    </thead>
    <tbody>
        <tr th:each="employee:${employees}">
            <td th:text="${employee.getId()}"></td>
            <td th:text="${employee.getLastName()}"></td>
            <td th:text="${employee.getEmail()}"></td>
            <td th:text="${employee.getGender() == 0 ? '女' : '男'}"></td>
            <td th:text="${employee.getDepartment()}"></td>
            <td th:text="${#dates.format(employee.getBirthday(), 'yyyy-MM-dd')}"></td>
            <td>
                <button class="btn btn-sm btn-primary">edit</button>
                <button class="btn btn-sm btn-danger">delete</button>
            </td>
        </tr>
    </tbody>
</table>
```

增删改

![img_24.png](../../img/2022/07/springboot_note/img_24.png)

新增表单
```html
<form th:action="@{/insertEmp}" th:method="post">
    <div class="form-group">
        <label>LastName</label>
        <input type="text" class="form-control" name="lastName">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input type="email" class="form-control" name="email">
    </div>
    <div class="form-group">
        <label>Gender</label>
        <div class="form-check form-check-inline">
            <input type="radio" class="form-check-input" name="gender" value="1">
            <label class="form-check-label">man</label>
        </div>
        <div class="form-check form-check-inline">
            <input type="radio" class="form-check-input" name="gender" value="2">
            <label class="form-check-label">women</label>
        </div>
    </div>
    <div class="form-group">
        <label>Department</label>
        <select name="department.id" id="dept" class="form-control">
            <option th:each="dept:${departments}" th:value="${dept.getId()}">[[${dept.getDepartName()}]]</option>
        </select>
    </div>
    <div class="form-group">
        <label>Birthday</label>
        <input type="date" class="form-control" name="birthday">
    </div>
    <button type="submit" class="btn btn-primary">insert</button>
</form>
```

修改表单
```html
<form th:action="@{/editEmp}" th:method="post">
    <div class="form-group" hidden>
        <label>Id</label>
        <input type="text" class="form-control" name="id" th:value="${emp.getId()}">
    </div>
    <div class="form-group">
        <label>LastName</label>
        <input type="text" class="form-control" name="lastName" th:value="${emp.getLastName()}">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input type="email" class="form-control" name="email" th:value="${emp.getEmail()}">
    </div>
    <div class="form-group">
        <label>Gender</label>
        <div class="form-check form-check-inline">
            <input type="radio" class="form-check-input" name="gender" value="1" th:checked="${emp.getGender() == 1}">
            <label class="form-check-label">man</label>
        </div>
        <div class="form-check form-check-inline">
            <input type="radio" class="form-check-input" name="gender" value="2" th:checked="${emp.getGender() == 2}">
            <label class="form-check-label">women</label>
        </div>
    </div>
    <div class="form-group">
        <label>Department</label>
        <select name="department.id" id="dept" class="form-control">
            <option th:each="dept:${departments}" th:value="${dept.getId()}"
                    th:selected="${emp.getDepartment() != null ? emp.getDepartment().getId() == dept.getId() : false}">[[${dept.getDepartName()}]]</option>
        </select>
    </div>
    <div class="form-group">
        <label>Birthday</label>
        <input type="date" class="form-control" name="birthday" th:value="${#dates.format(emp.getBirthday(), 'yyyy-MM-dd')}">
    </div>
    <button type="submit" class="btn btn-primary">save</button>
</form>
```

接口
```java
@Controller
public class EmployeeController {
    @Autowired
    private EmployeeService employeeService;

    @Autowired
    private DepartmentDao departmentDao;

    @GetMapping("/employees")
    public String getEmployees(Model model) {
        model.addAttribute("employees", employeeService.getEmployees());
        return "list";
    }

    @GetMapping("/insertPage")
    public String insertPage(Model model) {
        model.addAttribute("departments", departmentDao.getDepartments());
        return "add";
    }

    @PostMapping("/insertEmp")
    public String insertEmployee(Employee employee) {
        employeeService.insertEmployee(employee);
        return "redirect:/employees";
    }

    @GetMapping("/editPage/{id}")
    public String editPage(@PathVariable Integer id, Model model) {
        model.addAttribute("departments", departmentDao.getDepartments());
        model.addAttribute("emp", employeeService.getEmployeeById(id));
        return "edit";
    }

    @PostMapping("/editEmp")
    public String editEmployee(Employee employee) {
        employeeService.editEmployee(employee);
        return "redirect:/employees";
    }

    @GetMapping("/deleteEmp/{id}")
    public String deleteEmployee(@PathVariable Integer id) {
        employeeService.deleteEmployee(id);
        return "redirect:/employees";
    }
}
```

404页面处理

![img_25.png](../../img/2022/07/springboot_note/img_25.png)

## 4.1、JDBC
导包
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

配置
```yaml
spring: 
    datasource:
        username: root
        password: 123456
        url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
        driver-class-name: com.mysql.cj.jdbc.Driver
```

测试
```java
@SpringBootTest
class SpringbootStudyApplicationTests {

    @Autowired
    private DataSource dataSource;

    @Test
    void contextLoads() throws SQLException {
        System.out.println(dataSource.getClass());
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }

}
```

spring默认数据源：class com.zaxxer.hikari.HikariDataSource

JdbcTemplate是spring已经配置好的模板，拿来即用

CRUD
```java
@RestController
public class JdbcController {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @GetMapping("/userList")
    public List<Map<String, Object>> getUserList() {
        String sql = "select * from mybatis.user";
        List<Map<String, Object>> mapList = jdbcTemplate.queryForList(sql);
        return mapList;
    }

    @GetMapping("/addUser")
    public String addUser() {
        String sql = "insert into mybatis.user(id, name, pwd) values (4, '小明', '123456')";
        jdbcTemplate.update(sql);
        return "insert success";
    }

    @GetMapping("/updateUser/{id}")
    public String updateUser(@PathVariable("id") String id) {
        String sql = "update mybatis.user set name = ?, pwd = ? where id = " + id;
        jdbcTemplate.update(sql, new Object[]{"小红", "123456"});
        return "update success";
    }

    @GetMapping("/deleteUser/{id}")
    public String deleteUser(@PathVariable("id") String id) {
        String sql = "delete from mybatis.user where id = " + id;
        jdbcTemplate.update(sql);
        return "delete success";
    }

}
```

## 4.2、Druid
导包
```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.23</version>
    </dependency>
    
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

配置
```yaml
datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    # 数据源配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 1 from dual
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    # 配置监控统计拦截的filters, stat: 监控统计, log4j: 日志记录, wall: 防御sql注入
    filters: stat, log4j, wall
    maxPoolPreparedStatementPreConnectionSize: 20
    userGLobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

配置类注入bean及配置Druid监控页面
```java
@Configuration
public class DruidConfig {

    //注入bean
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druidDataSource() {
        return new DruidDataSource();
    }

    //后台监控配置，相当于原来的web.xml
    //因为SpringBoot内置了Servlet容器，所以没有web.xml，使用ServletRegistrationBean注入即可替代
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        //配置账号密码
        HashMap<String, String> initParam = new HashMap<>();
        initParam.put("loginUsername", "admin");
        initParam.put("loginPassword", "admin");
        //允许访问用户
        initParam.put("allow", "");
        bean.setInitParameters(initParam);
        return bean;
    }
}
```

## 4.2、整合Mybatis
导包
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

配置
```yaml
mybatis:
    type-aliases-package: com.wzc.springbootstudy.pojo #别名设置
    mapper-locations: classpath:mybatis/mapper/*.xml #mapper位置
```

xml、mapper、service、controller省略

## 5、SpringSecurity
AOP思想

Spring Security是Spring Boot底层安全模块默认的技术选型，使用Spring Security只需要导入spring-boot-starter-security
并进行少量配置即可

几个比较重要的类：
- WebSecurityConfigurerAdapter：自定义Security策略
- AuthenticationManagerBuilder：自定义认证策略
- @EnableWebSecurity：开启WebSecurity模式

SpringSecurity的两个主要目标是“认证（Authentication）”和“授权（Authorization）”

导包
````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
````
```xml
<!--thymeleaf整合spring security-->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```

自定义配置类
```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 授权
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //首页所有人都可以访问，特定的页面只有特定的角色才能访问
        http.formLogin().loginPage("/securityLogin").loginProcessingUrl("/login").defaultSuccessUrl("/security").and()//没有权限访问自动进入登录页
                .rememberMe().rememberMeParameter("rememberMe").and()//记住我功能，默认参数名为remember-me
                .logout().logoutSuccessUrl("/security").and()//注销页面
                .authorizeRequests()
                .antMatchers("/security").permitAll()
                .antMatchers("/securityLevel1/**").hasRole("vip1")
                .antMatchers("/securityLevel2/**").hasRole("vip2")
                .antMatchers("/securityLevel3/**").hasRole("vip3").and()
                .csrf().disable();
    }

    /**
     * 认证
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        //inMemoryAuthentication从内存中取数据，passwordEncoder密码编码规则，withUser添加用户
        BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
        auth.inMemoryAuthentication().passwordEncoder(bCryptPasswordEncoder)
                .withUser("wzc").password(bCryptPasswordEncoder.encode("123456")).roles("vip1", "vip2").and()
                .withUser("admin").password(bCryptPasswordEncoder.encode("admin")).roles("vip1", "vip2", "vip3").and()
                .withUser("guest").password(bCryptPasswordEncoder.encode("123456")).roles("vip3");

    }
}
```

index.html
```html
xmlns:sec="http://www.thymeleaf.org/extras/spring-security"
```

![img_26.png](../../img/2022/07/springboot_note/img_26.png)

![img_27.png](../../img/2022/07/springboot_note/img_27.png)

## 6、Shiro

### 6.1、快速开始

shiro架构中的几个对象：
- Subject：代表当前用户，与Subject的所有交互都会委托给SecurityManager
- SecurityManager：负责管理所有的Subject，是shiro的心脏，所有具体的交互都通过SecurityManager进行控制，
  且负责进行认证、授权、会话、及缓存的管理
- Authenticator：负责Subject认证逻辑
- Authorizer：授权器，控制用户是否有权进行相应操作
- Realm：安全试题数据源
- SessionManager：管理Session生命周期
- CacheManager：缓存控制器，管理用户、角色、权限等缓存
- Cryptography：密码模块

1. 导入依赖
```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.6.0</version>
</dependency>
```
2. 添加配置文件ini
```ini
[users]
# user 'root' with password 'secret' and the 'admin' role
root = secret, admin
# user 'guest' with the password 'guest' and the 'guest' role
guest = guest, guest
# user 'presidentskroob' with password '12345' ("That's the same combination on
# my luggage!!!" ;)), and role 'president'
presidentskroob = 12345, president
# user 'darkhelmet' with password 'ludicrousspeed' and roles 'darklord' and 'schwartz'
darkhelmet = ludicrousspeed, darklord, schwartz
# user 'lonestarr' with password 'vespa' and roles 'goodguy' and 'schwartz'
lonestarr = vespa, goodguy, schwartz

[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5
```
3. quickstart.java
```java
public class ShiroQuickStart {

    private static final transient Logger log = LoggerFactory.getLogger(ShiroQuickStart.class);

    public static void main(String[] args) {
        
        //通过ini配置文件获得SecurityManager实例
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        
        SecurityUtils.setSecurityManager(securityManager);

        //获得当前用户对象
        Subject currentUser = SecurityUtils.getSubject();

        //Session使用
        Session session = currentUser.getSession();
        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("Retrieved the correct value! [" + value + "]");
        }

        //用户登录
        if (!currentUser.isAuthenticated()) {
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
            token.setRememberMe(true);
            try {
                currentUser.login(token);
            } catch (UnknownAccountException uae) {
                log.info("There is no user with username of " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {
                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            } catch (AuthenticationException ae) {
                //unexpected condition?  error?
            }
        }

        //打印身份信息
        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");

        //校验角色
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }

        //校验权限
        if (currentUser.isPermitted("lightsaber:wield")) {
            log.info("You may use a lightsaber ring.  Use it wisely.");
        } else {
            log.info("Sorry, lightsaber rings are for schwartz masters only.");
        }
        
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }

        //注销
        currentUser.logout();

        System.exit(0);
    }
}
```

### 6.1、Shiro登录拦截

编写配置类
```java
@Configuration
public class ShiroConfig {
    /**
     * 创建自定义Realm对象
     * @return
     */
    @Bean
    public ShiroRealm shiroRealm() {
        return new ShiroRealm();
    }

    /**
     * 创建SecurityManager对象
     * @param shiroRealm
     * @return
     */
    @Bean
    public DefaultWebSecurityManager defaultWebSecurityManager(ShiroRealm shiroRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(shiroRealm);
        return securityManager;
    }

    /**
     * 创建ShiroFilterFactoryBean对象
     * @param defaultWebSecurityManager
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
        factoryBean.setSecurityManager(defaultWebSecurityManager);
        /**
         * 添加shiro内置过滤器
         * anon: 无需认证就可以访问
         * authc: 必须认证了才能访问
         * user: 必须拥有记住我才能访问
         * perms: 拥有对某个资源的权限才能访问
         * roles:拥有某个角色权限才能访问
         */
        Map<String, String> filterMap = new LinkedHashMap();
        filterMap.put("/shiro/add", "authc");
        factoryBean.setFilterChainDefinitionMap(filterMap);//设置内置过滤器
        factoryBean.setLoginUrl("/shiro/login");//设置登录url
        return factoryBean;
    }
}

```

自定义Realm对象
```java
public class ShiroRealm extends AuthorizingRealm {

    @Autowired
    UserService userService;

    /**
     * 授权
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("授权");
        User user = (User) principalCollection.getPrimaryPrincipal();
        //根据数据库中的权限进行授权
        if (!StringUtils.isEmpty(user.getPerms())) {
            SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
            info.addStringPermission(user.getPerms());
            return info;
        }
        return null;
    }

    /**
     * 认证
     * @param authenticationToken
     * @return
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("认证");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        User user = userService.queryByUserName(token.getUsername());
        if (ObjectUtils.isEmpty(user)) {
            throw new UnknownAccountException();//用户名不存在！
        }
        //密码校验由shiro完成
        return new SimpleAuthenticationInfo(user, user.getPwd(), user.getName());//第一个参数为授权方法中principalCollection.getPrimaryPrincipal()的值
    }
}
```

导入thymeleaf-extras-shiro
```xml
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

配置Bean整合thymeleaf
```java
@Configuration
public class ShiroConfig {
    /**
     * 创建ShiroDialect整合thymeleaf
     * @return
     */
    @Bean
    public ShiroDialect shiroDialect() {
        return new ShiroDialect();
    }
}
```

```html
<html xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
```

```html
<div shiro:guest="">
    <a th:href="@{/shiro/login}">login</a>
</div>
<div shiro:hasPermission="'user:add'">
    <a th:href="@{/shiro/add}">add</a>
</div>
<div shiro:user="">
    <a th:href="@{/shiro/logout}">logout</a>
</div>
```

## 7、Swagger

1. 导入依赖
```xml
<dependencies>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.9.2</version>
    </dependency>

    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.9.2</version>
    </dependency>
</dependencies>
```

2. 编写配置类
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Value("${swagger.enable}")
    private Boolean enable;

    @Bean
    public Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // .apis(RequestHandlerSelectors.none()) //不扫描所有接口
                // .apis(RequestHandlerSelectors.any()) //扫描所有接口
                .apis(RequestHandlerSelectors.basePackage("com.wzc.springbootstudy.controller")) //扫描包下的api接口
                // .apis(RequestHandlerSelectors.withClassAnnotation(RestController.class)) //扫描类上有RestController注解的接口
                // .apis(RequestHandlerSelectors.withMethodAnnotation(GetMapping.class)) //扫描方法上有GetMapping注解的接口
                // .paths(PathSelectors.ant("/shiro/**")) //扫描路径下的api接口
                .build()
                .enable(enable);
    }

    private ApiInfo apiInfo() {
        return new ApiInfo(
                "Swagger document",
                "description",
                "v1.0",
                "https://www.wzc.com",
                new Contact("wzc", "https://www.wzc.com", "714479133@qq.com"),
                "Apache 2.0",
                "https://www.apache.org/licenses/LICENSE-2.0",
                new ArrayList<>()
        );
    }
}
```

实体类中的注解
```java
@Data
@ApiModel("用户")
public class User implements Serializable {
    @ApiModelProperty("id")
    private Integer id;

    @ApiModelProperty("用户名")
    private String name;

    @ApiModelProperty("密码")
    private String pwd;

    @ApiModelProperty("年龄")
    private Integer age;

    @ApiModelProperty("邮箱")
    private String email;

    @ApiModelProperty("权限")
    private String perms;
}
```

效果

![img_28.png](../../img/2022/07/springboot_note/img_28.png)

接口类中的注解
```java
@Api(tags = "用户控制器")
@RestController
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @ApiOperation(value = "获取用户列表", notes = "需要注意的地方")
    @GetMapping("/queryUserList")
    public List<User> queryUserList() {
        return userMapper.queryUserList();
    }

}
```

效果

![img_29.png](../../img/2022/07/springboot_note/img_29.png)

## 8、异步任务

启动类上添加enable注解，开启异步
![img_30.png](../../img/2022/07/springboot_note/img_30.png)

异步方法
```java
@Service
public class AsyncService {

    @Async
    public void testAsync() {
        try {
            for (int i = 0; i < 10; i++) {
                System.out.println(i + 1);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

## 9、邮件任务

1. 导包
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

相关类：
- MailSenderAutoConfiguration
- MailProperties
- MailSenderJndiConfiguration

2. 配置
```yaml
mail:
    username: 714479133@qq.com
    password: keqpddottxodbcfg
    host: smtp.qq.com
    # qq邮箱需要开启加密验证
    properties:
        mail:
            smtp:
                ssl:
                    enable: true
```

3. 测试
```java
class Test {
    /**
     * 发送邮件
     */
    private void simpleMailSend() {
        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
        simpleMailMessage.setSubject("测试springboot邮件发送");
        simpleMailMessage.setText("测试");
        simpleMailMessage.setFrom("714479133@qq.com");
        simpleMailMessage.setTo("714479133@qq.com");
        javaMailSender.send(simpleMailMessage);
    }

    /**
     * 发送复杂邮件
     */
    private void mimeMailSend() throws MessagingException {
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
        mimeMessageHelper.setSubject("测试复杂邮件发送");
        mimeMessageHelper.setText("<p>html格式正文</p>", true);
        mimeMessageHelper.addAttachment("file1.jpg", new File("E:\\Code\\javaWorkSpace\\springboot-study\\note\\img.png"));
        mimeMessageHelper.setFrom("714479133@qq.com");
        mimeMessageHelper.setTo("714479133@qq.com");
        javaMailSender.send(mimeMessage);
    }
}
```

## 10、定时任务

启动类上添加@EnableScheduling注解开启定时任务

```java
@Service
public class ScheduledService {

    /**
     * cron表达式：秒 分 时 日 月 周
     */
    @Scheduled(cron = "0 0 12 * * ?")
    public void ScheduleTask() {
        System.out.println("每天12点执行一次！");
    }
}
```
