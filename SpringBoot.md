[toc]
# SpringBoot 基础
## 配置文件

初次创建`SpringBoot`项目之后，在`src/main/resources/`目录下存在配置文件 `application.properties`，一般不会使用，删掉创建
`application.yaml`文件

> `yaml`：专门用来写配置文件的语言，可以写普通的键值对、对象和数组，对空格的要求及其严格，一个空格表示一个层级关系，可以注入到配置类中
```yaml
# 普通的key-value
name: wyj

# 对象
student1:
  name: wyj
  age: 18

# 行内写法
student2: {name: wyj, age: 18}

# 数组
pets1:
  - cat
  - dog
  - pig

pets2: [cat,dog,pig]
```

## 使用 yaml 文件给实体类赋值

构建实体类 `Person` 和 `Dog` 类, 使用 `@Value` 可以为实体类的属性赋值, 如下 `Dog` 类
```java
@Component
public class Dog {
    @Value("旺柴")
    private String name;
    @Value("3")
    private Integer age;
    
}

@Component
public class Person {
    private String name;
    private Integer age;
    private boolean happy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
    
    //省略构造器、getter、setter方法
    //......
}
```

使用 `yaml` 文件对`Person`类进行赋值(不使用`@Value`) :

- 需要对 `Person` 类添加  `@ConfigurationProperties` 标签并与 `yaml` 文件中的 `person` 进行绑定
- `@ConfigurationProperties`: 将配置文件中的每一个属性的值, 映射到这个组件中
- 参数 `prefix = "person" `: 将配置文件中的 `person` 下面的属性一一对应


```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private boolean happy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
    
    //省略构造器、getter、setter方法
    //......
}
```

`yaml` 文件:

```yaml
person:
  name: wyj
  age: 18
  happy: true
  birth: 2020/12/12
  maps: {k1: v1, k2: v2}
  lists:
    - music
    - girl
    - sport
  dog:
    name: 旺柴
    age: 3

```

测试输出:
```
Person{name='wyj', age=18, happy=true, birth=Sat Dec 12 00:00:00 CST 2020, maps={k1=v1, k2=v2}, lists=[music, girl, sport], dog=Dog{name='wangchai', age=3}}
```

拓展:  

- 也可以使用 `@ProPertySource` 来加载指定的配置文件  
- `yaml` 文件中可以使用占位符号 `${}` 随机生成内容

```yaml
dog:
    name: wangchai_${person.hello:hello}
    age: 3
```

测试输出:
```
dog=Dog{name='wangchai_hello', age=3}
```

**`yaml`**和**`@ConfigurationProperties`** :

- 配置 `yaml` 和 `@ConfigurationProperties` 都可以获取到值,推荐 `yaml`
- 某个业务中, 只需要获取某个配置文件中的某个值, 可以使用 `@Value`
- 针对 `JavaBean` 来和配置文件进行映射, 使用 `yaml` 配置


## JSR303校验

**`JSR303`校验**: 用来进行数据校验

若上述代码中类 `Person` 中的属性 `name` 需要输入邮箱格式, 则可以在类 `Person` 上加 `@Validated`, 在 `name` 属性上加上校验注解 `@Email()` 

```java
@Validated
public class Person {

    @Email(message = "用户名格式错误,请输入邮箱格式")
    private String name;
    private Integer age;
    private boolean happy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
    
    //省略构造器、getter、setter方法
    //......
}
```

yaml 文件:
```yaml
person:
  name: wyj@qq.com
  
  #省略其它属性
  #......
```



常用的校验注解包含: 

| 注解名称             | 功能                                                         |
| -------------------- | ------------------------------------------------------------ |
| @Null                | 检查该字段是否为空                                           |
| @NotNull             | 不能为空                                                     |
| @NotEmpty            | 不能为空, 多用于检查 list 是否 size 为 0                     |
| @Max                 | 该字段的值只能小于或等于该值                                 |
| @Min                 | 该字段的值只能大于或者等于改值                               |
| @Past                | 检查该字段的日期是在过去                                     |
| @Future              | 检查该字段的日期是将来的日期                                 |
| @Eamil               | 检查是否为一个有效的邮箱                                     |
| @Size(min =, max = ) | 检查该字段的 size 是否在 min 和 max 之间, 可以是字符串、数组、集合、Map等 |
| @Pattern             | 被注解的元素必须是指定的正则表达式                           |

**以上代码在 SpringBoot-demo 项目中**

## 多环境配置及配置文件位置

`yaml` 文件可以写在 

1. `file:./config` : 项目根目录下 ( 项目名称下 ) 的 config 文件夹下
2. `file:./` : 项目根目录
3. `classpath:/config` : 类路径 ( java 文件夹下或 resources 文件夹下) 下的 config 文件夹下
4. `classpath:/` : 类路径下

优先级逐渐降低

通过 `yaml` 配置文件进行多环境配置, 如下:

`application.yaml`:
```yaml
server:
  port: 8080
spring:
  profiles:
    active: dev

---

# 开发环境
server:
  port: 8081
spring:
  profiles: dev

---

# 测试环境
server:
  port: 8082
spring:
  profiles: test
  
```

**测试代码在 SpringBoot-demo1 项目中**

# SpringBoot Web 开发

## 静态资源导入

创建项目 `SpringBoot-Web-demo1`, 项目测试中`http://localhost:8080//hello` 不可使用, 暂时使用
`http://127.0.0.1:8080//hello` 启动 tomcat 访问 controller, 第二天后 localhost 可以使用了

测试 `controller` 代码: 
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        return "hello, world!";
    }
}
```

静态资源可以放在类路径 `resources` 下的 `static` 、`public` 、  `resources` 文件夹下, 文件夹下的内容通过 `http://localhost:8080/**` 即可访问, 例如 `static` 文件夹下有 `1.js`, 则直接通过 `http://localhost:8080/1.js` 便可访问成功

访问优先级别: `resources`> `static` > `public`

一般 `public` 放公共资源, `static` 放静态资源如图片, `resources` 下放上传文件


## 首页和图标定制

定制首页的 `index.html` 时, 可以将 `index.html` 文件放在类路径 resources 下的 `resources` 、 `static` 、 `public` 文件夹下, 此时便可以直接通过 `http://localhost:8080` 直接访问到 `index.html`链接

## 模板引擎 Thymeleaf

新建项目 `SpringBoot-Web-demo2`, 创建项目时候将 `Dependencies`-> `Template Engines` -> `Thymeleaf` 选上

**`Thymeleaf`** 用来开发 `Web`  和独立环境项目的服务器端的Java模版引擎, 代替传统的 `Jsp` 文件, 使用 `Thymeleaf` 的文件必须放在类路径 resources 下的 `templates` 文件夹下, `templates` 文件夹下的内容不能通过 
`http://localhost:8080` 直接访问, 必须要通过 controller 进行跳转

实现首页的跳转, controller 层源码:
```java
@Controller
public class IndexController {
    @RequestMapping("/index")
    public String index() {
        return "index";
    }
}
```

`templates` 文件夹下的 `index.html` 源码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    使用thymeleaf
</body>
</html>
```

html 中使用模板引擎 thymeleaf 时, 首先要在 html 文件引入 thymeleaf 模板引擎, 首先要在 html 文件中引入头文件

```html
xmlns:th="http://www.thymeleaf.org" 
<html lang="en" xmlns:th="http://www.thymeleaf.org" >
```

引入头文件之后, 可以绑定 html 文件里的任何元素, 使用 **`th:元素名`** 的方式, 源码如下:


```java
@Controller
public class IndexThymeleafController {

    @RequestMapping("/index")
    public String index(Model model) {
        model.addAttribute("msg", "hello, thymeleaf");
        return "index";
    }

}
```


```html
<!DOCTYPE html >
<html lang="en" xmlns:th="http://www.thymeleaf.org" >
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    使用thymeleaf
    <div th:text="${msg}"></div>
</body>
</html>
```

## Thymeleaf 语法

[`Thymeleaf`官方文档参考](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.pdf)

**Thymeleaf 语法如下**:

| 属性          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| `th:text`     | 文本替换                                                     |
| `th:utext`    | 支持 html 的文本替换                                         |
| `th:value`    | 属性赋值                                                     |
| `th:each`     | 遍历循环元素                                                 |
| `th:if`       | 判断条件, 类似的还有th:unless、th:switch、th:case            |
| `th:insert`   | 代码块引入,类似的还有th:replace, th:include 常用于公共代码块提取的场景 |
| `th:fragment` | 定义代码块, 方便被 th:insert 引用                            |
| `th:object`   | 声明变量, 一般和*{} 一起配合使用                             |
| `th:attr`     | 设置标签属性, 多个属性可以用逗号分隔                         |

源码演示:

```html
<!DOCTYPE html >
<html lang="en" xmlns:th="http://www.thymeleaf.org" >
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    使用thymeleaf
    <div th:text="${msg}"></div>
    <h1 th:each="user:${users}" th:text="${user}"></h1>
</body>
</html>
```


```java
@Controller
public class IndexThymeleafController {
    @RequestMapping("/index")
    public String index(Model model) {
        model.addAttribute("msg", "hello, thymeleaf");
        model.addAttribute("users", Arrays.asList("wyj", "mary"));
        return "index";
    }
}
```

**Thymeleaf 表达式如下:**


| 表达式   | 含义           |
| -------- | -------------- |
| `${...}` | 变量表达式     |
| `@{...}` | 链接表达式     |
| `#{...}` | 消息表达式     |
| `~{...}` | 代码块表达式   |
| `*{...}` | 选择变量表达式 |


## 自定义 MVC  配置原理

(1)自定义SpringMVC 功能时, 需要自定义一个类并实现接口 `WebMvcConfigurer`

(2)自定义视图解析器需要创建一个类并实现接口 `ViewResolver`

源码如下:
```java
/**
 * 自定义SpringMVC功能, 扩展 SpringMVC
 */
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    //取自定义视图解析器作为一个 Bean
    @Bean
    public ViewResolver myViewResolver() {
        return new MyViewResolver();
    }

    //自定义视图解析器
    public static class MyViewResolver implements ViewResolver {
        @Override
        public View resolveViewName(String viewName, Locale locale) throws Exception {
            return null;
        }
    }
}
```


# 员工管理系统

## 员工管理系统 : 准备工作

### Dao 层和 Pojo 层源码

创建新的项目 `SpringBoot-EmployeeMana-01`, Pojo 层创建类 `Department` 和类 `Employee`, Dao 层创建类 `DepartmentDao` 和类 `EmployeeDao`, 并模拟数据库生成数据, 源码如下 :


```java
/**
 * 部门表
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Department {
    private Integer id;
    private String departmentName;
}

/**
 * 员工表
 */
@Data
@NoArgsConstructor
public class Employee {

    private Integer id;
    private String lastName;
    private String email;
    private Integer gender;//boy-1 girl-0
    private Department department;
    private Date birth;

  	public Employee(Integer id, String lastName, String email, Integer gender, Department department) {
        this.id = id;
        this.lastName = lastName;
        this.email = email;
        this.gender = gender;
        this.department = department;
        // 默认创建日期
        this.birth = new Date();
    }
}
```


```java
@Repository
public class DepartmentDao {
    //模拟数据库的数据
    private static Map<Integer, Department> departments = null;

    static {
        departments = new HashMap<Integer, Department>();

        departments.put(101, new Department(101, "教学部"));
        departments.put(102, new Department(102, "市场部"));
        departments.put(103, new Department(103, "教研部"));
        departments.put(104, new Department(104, "运营部"));
        departments.put(105, new Department(105, "后勤部"));
    }

    //获取所有部门信息
    public Collection<Department> getDepartment() {
        return departments.values();
    }

    //通过 id 获取部门名称
    public Department getDepartmentById(Integer id) {
        return departments.get(id);
    }
}

@Repository
public class EmployeeDao {
    //模拟数据库的数据
    private static Map<Integer, Employee> employees = null;

    @Autowired
    private DepartmentDao departmentDao;

    static {
        employees = new HashMap<Integer, Employee>();

        employees.put(1001, new Employee(1001, "AA", "1013201176@qq.com", 1, new Department(101, "教学部")));
        employees.put(1002, new Employee(1002, "BB", "2013201176@qq.com", 0, new Department(102, "市场部")));
        employees.put(1003, new Employee(1003, "CC", "3013201176@qq.com", 0, new Department(103, "教研部")));
        employees.put(1004, new Employee(1004, "DD", "4013201176@qq.com", 1, new Department(104, "运营部")));
        employees.put(1005, new Employee(1005, "EE", "5013201176@qq.com", 1, new Department(105, "后勤部")));
    }

    //员工主键自增
    private static Integer initId = 1006;
    // 增加一个员工
    public void save(Employee employee) {
        if (employee.getId() == null) {
            employee.setId(initId++);
        }
        // 该行代码表明每次增加的员工一定有相对于的部门
        employee.setDepartment(departmentDao.getDepartmentById(employee.getDepartment().getId()));
        employees.put(employee.getId(), employee);
    }

    // 查询全部员工信息
    public Collection<Employee> getAll() {
        return employees.values();
    }

    // 通过 id 查询员工
    public Employee getEmployeeById(Integer id) {
        return employees.get(id);
    }

    // 删除员工信息通过 id
    public void delete(Integer id) {
        employees.remove(id);
    }
}
```


### Lombok 的使用

**Lombok**在使用之前需要进行安装, 参考链接[IDEA如何安装lombok插件（在线和离线两种方式）](https://blog.csdn.net/qq_36043458/article/details/103166297)

**Lombok** 可以通过注解的方式, 在编译时自动为属性生成构造器, getter/setter, equals, hashcode, toString方法, 参考链接 : [Java开发中用到的，lombok是什么？](https://www.zhihu.com/question/42348457), 部分 lombok 标签含义:

| 标签                  | 含义                                                         |
| --------------------- | ------------------------------------------------------------ |
| `@NoArgsConstructor`  | 用在类上, 自动生成无参构造函数                               |
| `@AllArgsConstructor` | 用在类上, 自动生成使用所有参数的构造函数                     |
| `@Data`               | 注解在类上，相当于同时使用了@ToString、@EqualsAndHashCode、@Getter、@Setter和@RequiredArgsConstrutor这些注解 |

SpringBoot 项目中使用 Lombok 需要在 pom 文件中引入依赖 : 

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```


### 静态资源引入

静态资源模板可以在网上搜索 `Bootstrap` 模板下载使用, `html` 文件放在 templates 文件夹下, `css`、`js`、`img` 放在 static 文件夹下

## 员工管理系统

### 员工管理系统 : 首页实现

**首页配置 : 所有页面的静态资源都需要使用 Thymeleaf 接管**

首页的实现需要一个 `controller`, 源码如下:
```java
@Controller
public class IndexController {
    @RequestMapping({"/", "/index.html"})
    public String index() {
        return "index";
    }
}
```

或者在自定义 `MVC` 配置类 `MyMvcConfig` 中实现 :

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("").setViewName("index");
    }
}

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
        registry.addViewController("/index.html").setViewName("index");
    }
}
```

之后便可以删掉 controller 类 `IndexController`, 对 `index.html` 等文件里面的元素标签内容进行修改, 修改为 Thymeleaf 格式 :

**第一步** : 要在 `html` 文件中引入 `Thymeleaf` 头文件

```html
xmlns:th="http://www.thymeleaf.org"
```
**第二步** : 修改 `html` 文件中非 `Thymeleaf` 语法的内容 :

`html` 中本地链接内容使用 `th:href` 修改, 并使用 `@{}` 进行取值

> 注意: 使用 `@{/**}` 的形式代表项目的 `class` 目录, 即项目的根目录。找 `css` 文件夹下的 `.css` 文件, 可以直接写成 `@{/css/bootstrap.min.css}`,  `resources` 文件夹下的 `static` 、`templates` 都是静态目录, 都不用写

```html
<link href="asserts/css/bootstrap.min.css" rel="stylesheet">
```
修改为 :

```html
<link th:href="@{/css/bootstrap.min.css}" rel="stylesheet">
```
---

`src` 修改为 `th:src` :

```html
<img class="mb-4" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
```
修改为 :
```html
<img class="mb-4" th:src="@{/img/bootstrap-solid.svg}" alt="" width="72" height="72">
```
---
非本地链接不要修改 :
```html
<a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
```
---
可以在 yaml 文件添加以下内容 :

```yaml
# 项目虚拟路径
server:
  servlet:
    context-path: /wyj
```
修改项目虚拟路径, 之后访问项目的时候则需要在 `http://localhost:8080/`
加上 `wyj`,  `http://localhost:8080/wyj`

之后项目启动会自动拼接 :

```html
<link href="/wyj/css/bootstrap.min.css" rel="stylesheet">
```

### 员工管理系统 : 国际化

参考视频链接 : [员工管理系统 : 国际化](https://www.bilibili.com/video/BV1PE411i7CV?p=22)


### 员工管理系统 : 登录功能实现

登录功能的实现, 登录之后 `index.html` 要跳转到一个 controller, 修改代码 :

跳转 action 使用 `th:action` 修改 :


```html
<form class="form-signin" action="dashboard.html">
```
修改为 :

```html
<form class="form-signin" th:action="@{/user/login}">
```

`@{/user/login}`为类 `LoginController`中写的登录跳转链接。将前端 `index.html` 输入的用户名 `name="username"` 和密码 `name="password"` 绑定到类 `LoginController` 的 login() 方法参数中(使用标签`@RequestParam`), 之后跳转到类 `LoginController` 进行页面的分发处理 :


```html
<input type="text" name="username" class="form-control" placeholder="Username" required="" autofocus="">
			
<input type="password" name="password" class="form-control" placeholder="Password" required="">
```

```java
@Controller
public class LoginController {
    @RequestMapping("/user/login")
    //@ResponseBody
    public String login(
            @RequestParam("username") String username,
            @RequestParam("password") String password,
            Model model) {
        // 具体业务
        if (!StringUtils.isEmpty(username) && password.equals("123456")) {
            return "dashboard";
        } else {
            // 用户名密码错误
            model.addAttribute("msg", "用户名或者密码错误！");
            return "index";
        }
    }
}
```

用户名或者密码输入正确, 跳转到 `dashboard.html` 页面,
用户名或者密码输入错误, 跳回到 `index.html` 页面, 并将错误信息显示出来, 错误信息显示需要在 `index.html` 文件中添加如下代码 :

```html
<p style="color: #ff0000" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

以上代码表示如果 `msg` 不为空, 则将 `msg` 的信息显示到登录界面, `Thymeleaf` 中存在很多工具类, 如 `#strings.isEmpty(msg)` 表示 msg 消息是否为空, 具体参考[Thymeleaf 表达式工具类](https://blog.csdn.net/ryuenkyo/article/details/103291168)

登录跳转成功之后, 跳转到 `dashboard.html` 页面, 会发现浏览器网址链接显示 : 
```http
http://localhost:8080/wyj/user/login?username=wyj&password=123456
```
为了更符合实际项目开发, 隐藏 `user/login?username=wyj&password=123456
`, 可以做一个简单的映射关系, 在类 `MyMvcConfig` 的 `addViewControllers()` 方法添加一行映射关系代码 :

```java
registry.addViewController("main.html").setViewName("dashboard");
```
以上代码表示如何如果访问 `main.html`, 则会跳转到 `dashboard.html` 页面中, 在将类 `LoginController` 跳转到 `dashboard.html` 页面的代码修改 :

```java
if (!StringUtils.isEmpty(username) && password.equals("123456")) {
    return "dashboard";
}
```
修改为 :

```java
if (!StringUtils.isEmpty(username) && password.equals("123456")) {
    //网页重定向
    //return "dashboard"; 
    return "redirect:/main.html";
} 
```

最终登录成功跳转到 `dashboard.html` 页面, 浏览器显示的链接为 :

```http
http://localhost:8080/wyj/main.html
```

### 员工管理系统 : 登录拦截器

登录拦截器需要实现的功能为 : 员工在登录之后进行拦截, 判断用户是否有权限登录, 如果有则进行跳转, 否则退回到登录页面, 并提示用户没有权限登录。因此需要在 `controller` 的类 `LoginController` `login()` 方法中拿到用户信息, 并交给拦截器类进行用户权限的判断, 使用 Session 可以在多个页面之间进行信息传递


```java
if (!StringUtils.isEmpty(username) && password.equals("123456")) {
    session.setAttribute("loginUser", username);
    //获取用户信息保存在 session 中,在拦截器类中拿到信息进行权限判断
    session.setAttribute("loginUser", username);
    //return "dashboard";
    return "redirect:/main.html";
}
```
拦截器的实现需要在 `config` 文件夹下创建类 `LoginHandlerInterceptor`, 实现接口 `HandlerInterceptor`, 并重写 `preHandle()` 方法


```java
public class LoginHandlerInterceptor implements HandlerInterceptor {
    // preHandle() 在请求方法之前执行,返回值为boolean类型,true表示请求继续执行,false表示请求结束,postHandle()在请求处理完成,dispatcherServlet返回视图后执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //登录成功之后, 拿到用户的 session 信息
        Object loginUser = request.getSession().getAttribute("loginUser");

        if (loginUser == null) { //进行权限判断
            request.setAttribute("msg", "没有权限, 请先登录");
            // 请求的分发, 退回到登录界面
            request.getRequestDispatcher("/index.html").forward(request, response);
            return false;
        } else {
            return true;
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception { }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception { }
}
```

拦截器类 `HandlerInterceptor` 需要配置在 `Bean` 里面注册, 在 `MVC` 自定义配置类 `MyMvcConfig` 中重写方法 `addInterceptors()` 

```java
//将自定义拦截器注册到 Bean 里面
@Override
public void addInterceptors(InterceptorRegistry registry) {
    // addPathPatterns("/**") : 拦截所有请求
    // excludePathPatterns() : 不需要拦截的请求
    registry.addInterceptor(new LoginHandlerInterceptor())
            .addPathPatterns("/**")
            .excludePathPatterns("/index.html", "/", "/user/login", "/css/*", "/img/**", "/js/**");
}
```

登录成功之后, 在主界面 `dashboard.html` 显示登录的用户名, 修改 `dashboard.html` 以下内容 :

```html
<a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Company name</a>
```
修改为 :

```html
<a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">[[${session.loginUser}]]</a>
```

注意 : `html` 中可以使用 `[[${session.loginUser}]]` 的方式取值

### 员工列表展示

#### 后台编写

员工列表的展示首先需要在 `controller` 层写一个类 `EmployeeController` :

```java
@Controller
public class EmployeeController {

    @Autowired
    EmployeeDao employeeDao;

    @RequestMapping("/emps")
    public String list(Model model) {
        // ctrl+alt+v 自动补全函数返回值
        Collection<Employee> employees = employeeDao.getAll();
        model.addAttribute("emps", employees);
        return "emp/list";
    }
}
```

#### th:fragment 提取公共页面

- 模板中，经常希望从其他模板中包含⼀些部分，如⻚眉，⻚脚，公共菜单等部分，为了做到这⼀点，`Thymeleaf` 可以使⽤ `th:fragment`  属性来定义被包含的模版⽚段，以供其他模版包含

- 使用 `th:fragment` 定义了需要 copy 的片段之后，可以使⽤ `th:insert` 或 `th:replace` 属性包含进需要的页面中

使用项目 `SpringBoot-EmployeeMana-02` 提取公共页面 : **侧边栏和头部导航栏内容**

![image-20211207111204341](SpringBoot 基础.assets/image-20211207111204341.png)

**员工管理前端页面跳转地址的修改：**

**1、项目需求**：点击 `员工管理` ，跳转到`list.html` 页面显示当前员工的信息

- 修改 `dashboard.html` 和 `list.html` 相应内容

  ```html
  <li class="nav-item">
     <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users">
           <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
           <circle cx="9" cy="7" r="4"></circle>
           <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
           <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
        </svg>
        Customers
     </a>
  </li>
  ```

  修改为：

  ```html
  <li class="nav-item">
     <a class="nav-link" th:href="@{/emps}">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users">
           <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
           <circle cx="9" cy="7" r="4"></circle>
           <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
           <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
        </svg>
        员工管理
     </a>
  </li>
  ```

  以上代码主要修改 `href` 部分，使用 `th:href` 接管

**2、抽取公共的代码部分**（`list.html`和`dashboard.html`）：侧边栏和顶部导航栏

- `dashboard.html`页面，使用 `th:fragment` 抽取代码公共部分内容

  ```html
  <!--顶部导航栏-->
  <nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" >
      <!--...-->
  </nav>
  
  <!--侧边栏-->
  <nav class="col-md-2 d-none d-md-block bg-light sidebar">
     <!--...-->
  </nav>
  ```

  修改为：

  ```html
  <!--顶部导航栏-->
  <nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" th:fragment="topbar">
      <!--...-->
  </nav>
  
  <!--侧边栏-->
  <nav class="col-md-2 d-none d-md-block bg-light sidebar" th:fragment="sidebar">
     <!--...-->
  </nav>
  ```

- `list.html` 页面

  使用 `dashboard.html` 抽取的公共部分内容来组建`list.html` 页面的侧边栏和顶部导航栏
  
  `list.html` 页面的顶部导航栏
  
  ```html
  <nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" >
      <!--...-->
  </nav>
  ```
  
  使用代码
  
  ```html
  <div th:insert="~{dashboard::topbar}"></div>
  ```
  
  代替，`list.html` 页面的侧边栏
  
  ```html
  <nav class="col-md-2 d-none d-md-block bg-light sidebar">
     <!--...-->
  </nav>
  ```
  
  使用代码
  
  ```html
  <div th:insert="~{dashboard::sidebar}"></div>
  ```
  
  代替，以上代码使用到 `Thymeleaf` 中的 `th:insert` 语法

**3、进一步抽取公共的代码**

公共代码部分侧边栏和头部导航栏内容可以单独抽取出来，放在一个单独的文件中

- 在`templates`目录下面创建`commons`目录，在`commons`目录下面创建`commons.html`放公共代码

- 将 `dashboard.html` 中的侧边栏和头部导航栏代码提取出来放在 `commons.html` 中

- 使用 `th:replace` 在 `dashboard.html` 中补全公共代码部分，此时的公共代码部分在 `commons` 文件夹下的`commons.html` 下

  ```html
  <!--顶部导航栏-->
  <div th:replace="~{commons/commons::topbar}"></div>
  
  <!--侧边栏-->
  <div th:replace="~{commons/commons::sidebar}"></div>
  ```

  `list.html` 补全方法与 `dashboard.html` 一样， **`th:replace` 和 `th:insert` 效果一样**

### 侧边栏点中高亮功能

在侧边栏中，鼠标点中某一选项，该选项会高亮显示，如下图所示：

![image-20211207172825857](SpringBoot 基础.assets/image-20211207172825857.png)



 `commons.html` 中的代码

```html
<a class="nav-link active" th:href="@{/main.html}">
```

`active` 表示高亮状态，当点击`首页` 选项的时候，调用 `dashboard.html` 中代码，此时可以在 `dashboard.html` 调用侧边栏代码部分传递一个参数，代码如下：

```html
<div th:replace="~{commons/commons::sidebar}"></div>
```

修改为：

```html
<div th:replace="~{commons/commons::sidebar(active='main.html')}"></div>
```

同理，`list.html` 代码修改如下：

```html
<!--侧边栏-->
<div th:insert="~{commons/commons::sidebar(active='list.html')}"></div>
```

`commons.html` 接收该到参数 `active` 进行侧边栏显示的时候，使用 **`Thymeleaf` 三元运算符：if-then-else:（if）?（then）:（else）**判断高亮显示哪一个鼠标点击事件

在  `commons.html` 中接受参数并进行判断代码修改如下：

```html
<a class="nav-link active" th:href="@{/main.html}">
```

修改为：

```html
<a th:class="${active=='main.html'?'nav-link active':'nav-link'}" th:href="@{/main.html}">
```

```html
<a class="nav-link active" th:href="@{/emps}">
```

修改为：

```html
<a th:class="${active=='list.html'?'nav-link active'?'nav-link'}" th:href="@{/emps}">
```

**注意要使用 `Thymeleaf` 中的 `th:class` 替换 `class`** ，以上代码整体片段如下：

```html
<!--侧边栏-->
<nav class="col-md-2 d-none d-md-block bg-light sidebar" th:fragment="sidebar">
    <div class="sidebar-sticky">
        <ul class="nav flex-column">
            <li class="nav-item">
               <a th:class="${active=='main.html'?'nav-link active':'nav-link'}" th:href="@{/main.html}">

                  	.............
                    首页 <span class="sr-only">(current)</span>
                </a>
            </li>
            .............
            <li class="nav-item">
            	<a th:class="${active=='list.html'?'nav-link active':'nav-link'}" th:href="@{/emps}">

                    .............
                    员工管理
                </a>
            </li>
            .............
        </ul>
        .............
    </div>
</nav>
</html>
```

### 员工列表信息循环显示

员工列表循环显示修改 `list.html` 文件相应内容，`<thead>` 为表头，`<tbody>`  为表数据，代码如下：

```html
<main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
   <h2>Section title</h2>
   <div class="table-responsive">
      <table class="table table-striped table-sm">
         <thead>
            <tr>
               <th>id</th>
               <th>lastName</th>
               <th>email</th>
               <th>gender</th>
               <th>department</th>
               <th>birth</th>
               <th>操作</th>
            </tr>
         </thead>
         <tbody>
            <tr th:each="emp:${emps}">
               <td th:text="${emp.getId()}"></td>
               <td th:text="${emp.getLastName()}"></td>
               <td th:text="${emp.getEmail()}"></td>
               <td th:text="${emp.getGender()==0?'女':'男'}"></td>
               <td th:text="${emp.department.getDepartmentName()}"></td>
               <td th:text="${#dates.format(emp.getBirth(),'yyyy-MM-dd HH:mm:ss')}"></td>
               <td>
                  <button class="btn btn-sm btn-primary">编辑</button>
                  <button class="btn btn-sm btn-danger">删除</button>
               </td>
            </tr>

         </tbody>
      </table>
   </div>
</main>
```

注意： `Thymeleaf` 中的定义了许多对日期格式进行设置的方法，例如：

```html
${#dates.format(date,'dd/MMM/yyyy HH:mm')}
```

### 添加员工信息

员工信息的添加需要单独编写一个`Button`按钮，[`BootStrap` 官方网站](https://getbootstrap.com/docs/5.1/components/accordion/)   中提供了前端界面各类组件的代码，直接在其官网寻找所需要的组件代码拿来用即可

流程：

- 当点击`添加`按钮时，跳转到员工信息添加界面 `add.html`，界面 `add.html`的头部导航栏和侧边栏仍然保持不变，因此需要在原有代码基础上创建一个添加界面 `add.html`，该界面需要一个 Form 表单，用来添加员工信息。
- 在创建的 Form 表单中添加完员工信息之后，提交到一个 `Controller`，该`Controller`将表单信息进行处理，添加到员工列表中去
- 该`Controller`最终返回全部员工信息展示到`list.html`界面

#### 添加员工信息按钮代码

在 `list.html` 中找到 `Section title` 修改为一个 `添加` 按钮

```html
<h2>Section title</h2>
```

修改为：

```html
<h2>
    <a type="button" class="btn btn-secondary" th:href="@{/emp}">添加</a>
</h2>
```

当点击`添加`按钮时，可以将请求发送到对应的`Controller`

![image-20211208164423067](SpringBoot 基础.assets/image-20211208164423067.png)

![image-20211208165615320](SpringBoot 基础.assets/image-20211208165615320.png)

#### 后台跳转 Controller 代码

使用 RestFul 分格来进行资源映射，在类`EmployeeController`编写后台`Controller`代码如下：

```java
@GetMapping("/emp")
public String toAddPage(Model model) {
    Collection<Department> departments = departmentDao.getDepartment();
    model.addAttribute("departments",departments);
    return "emp/add";
}
```

`toAddPage()`进行请求分发，将视图员工信息添加页面`add.html`显示到前端

![image-20211209164810639](SpringBoot 基础.assets/image-20211209164810639.png)

#### 创建添加员工信息代码

- 员工信息添加`add.html` 代码如下：

  ```html
  <main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
      <form th:action="@{/emp}" method="post">
          <div class="form-group">
              <label>姓名</label>
              <input type="text" name="lastName" class="form-control">
          </div>
          <div class="form-group">
              <label>邮箱</label>
              <input type="email" name="email" class="form-control"  >
          </div>
          <div class="form-group">
              <label>性别</label><br>
              <div class="form-check form-check-inline">
                  <input class="form-check-input" type="radio" name="gender" value="1">
                  <label class="form-check-label">男</label>
              </div>
              <div class="form-check form-check-inline">
                  <input class="form-check-input" type="radio" name="gender" value="0">
                  <label class="form-check-label">女</label>
              </div>
          </div>
          <div class="form-group">
              <label>部门</label>
              <select class="form-control" name="department.id">
                  <option th:each="dept:${departments}" th:text="${dept.getDepartmentName()}" th:value="${dept.getId()}">1</option>
              </select>
          </div>
          <div class="form-group">
              <label>生日</label>
              <input type="text" name="birth" class="form-control" placeholder="2020/07/25 18:00:00">
          </div>
          <button type="submit" class="btn btn-primary">添加</button>
      </form>
  </main>
  ```
  
  `add.html`中，每一个标签设置一个`name`属性，`name`属性值和员工类`Employee`定义的属性字段名称需要一样，才可以一一映射，员工类`Employee`如下
  
  ```java
  @Data
  @NoArgsConstructor
  public class Employee {
      private Integer id;
      private String lastName;
      private String email;
      private Integer gender;//boy-1 girl-0
      private Department department;
      private Date birth;
      //...
  }
  ```
  
  属性`department`是一个对象，类`EmployeeDao`中的`save()`方法每次增加一个新的员工信息时，该员工的部门名称是通过部门的`id`得到的，见`save()`方法如下代码：
  
  ```java
  // 该行代码表明每次增加的员工一定有相对应的部门
  employee.setDepartment(departmentDao.getDepartmentById(employee.getDepartment().getId()));
  ```
  
  因此在`add.html`中，需要拿到部门的`id`传递到后台代码，`add.html`中通过代码`th:value="${dept.getId()}"`得到部门`id`，映射到`name`属性`department.id`上，具体代码如下：
  
  ```html
  <div class="form-group">
      <label>部门</label>
      <select class="form-control" name="department.id">
          <option th:each="dept:${departments}" th:text="${dept.getDepartmentName()}" th:value="${dept.getId()}">1</option>
      </select>
  </div>
  ```

#### 显示最终员工信息代码

员工信息通过`add.html`添加之后，点击`添加`按钮，会将员工信息通过`Post`请求的方式传递到后台`Controller`代码，映射`URL`仍然为`/emp`，体现了 RestFul 风格，后台`Controller`代码将新添加的员工信息保存在员工信息列表中，并展示到前端显示，`Controller`代码如下：

```java
@Controller
public class EmployeeController {
    @Autowired
    DepartmentDao departmentDao;
    
   	//...省略部分代码
    
    @RequestMapping("/emps")
    public String list(Model model) {
        // ctrl+alt+v 自动补全函数返回值
        Collection<Employee> employees = employeeDao.getAll();
        model.addAttribute("emps", employees);
        return "emp/list";
    }
    
    @PostMapping("/emp")
    public String addEmp(Employee employee) {
        employeeDao.save(employee);
        return "redirect:/emps";
    }
}
```

### 日期格式修改

`Spring`中严格限制了日志格式的书写方式，默认是`yyyy/MM/dd`表达方式，如果写成`yyyy-MM-dd`，程序则会报错：

![image-20211210095647603](SpringBoot 基础.assets/image-20211210095647603.png)

![image-20211210095714892](SpringBoot 基础.assets/image-20211210095714892.png)



![image-20211210095733663](SpringBoot 基础.assets/image-20211210095733663.png)

需要**在 `application.yaml` 文件中添加配置来修改日期格式**

```xml
#日期格式
spring:
  mvc:
    format:
      date: yyyy-MM-dd
```

### 修改员工信息

#### 提交按钮代码

点击`编辑`按钮，跳转到后台`Controller`

```html
<a type="button" class="btn btn-sm btn-primary" th:href="@{/emp/}+${emp.getId()}">编辑</a>
```

![image-20211210104340318](SpringBoot 基础.assets/image-20211210104340318.png)

#### 后台Controller代码

后台`Controller`拿到需要编辑的员工`id`，查询到该员工的信息以及所有部门信息，并将请求分发到员工信息更新页面`update.html`

```java
@GetMapping("/emp/{id}")
public String toUpdateEmp(@PathVariable("id") Integer id, Model model) {
    //查出原来的数据
    Employee employee = employeeDao.getEmployeeById(id);
    model.addAttribute("emp", employee);
    //查出所有部门信息
    Collection<Department> departments = departmentDao.getDepartment();
    model.addAttribute("departments",departments);
    return "emp/update";
}
```

#### 员工信息修改前端

员工信息更新页面先将当前员工的所有信息展示到界面上，姓名、邮箱等`text`属性使用`th:value`显示，性别等`radio`属性使用`th:checked`显示，部门属性（下拉框`select`属性）使用`th:selected`显示

```html
<form th:action="@{/updateEmp}" method="post">
    <!--隐藏域-->
    <input type="hidden" name="id" th:value="${emp.getId()}">
   
    <div class="form-group">
        <label>姓名</label>
        <input th:value="${emp.getLastName()}" type="text" name="lastName" class="form-control">
    </div>

    <div class="form-group">
        <label>邮箱</label>
        <input th:value="${emp.getEmail()}" type="email" name="email" class="form-control"  >
    </div>
    
    <div class="form-group">
        <label>性别</label><br>
        <div class="form-check form-check-inline">
            <input th:checked="${emp.getGender()==1}" class="form-check-input" type="radio" name="gender" value="1">
            <label class="form-check-label">男</label>
        </div>
        <div class="form-check form-check-inline">
            <input th:checked="${emp.getGender()==0}" class="form-check-input" type="radio" name="gender" value="0">
            <label class="form-check-label">女</label>
        </div>

    </div>
    <div class="form-group">
        <label>部门</label>
        <select class="form-control" name="department.id">
            <option th:selected="${dept.getId()==emp.getDepartment().getId()}" th:each="dept:${departments}" th:text="${dept.getDepartmentName()}" th:value="${dept.getId()}">1</option>
        </select>
    </div>
    
    <div class="form-group">
        <label>生日</label>
        <input th:value="${#dates.format(emp.getBirth(), 'yyyy-MM-dd HH:mm:ss')}" type="text" name="birth" class="form-control" placeholder="2020/07/25 18:00:00">
    </div>
    
    <button type="submit" class="btn btn-primary">修改</button>
</form>
```

注意：

- 属性`生日`使用 `Thymeleaf`中 `#dates.format()`对日期格式进行了转化

- 使用隐藏域`type="hidden"`来获取员工的`id`信息，使其不显示到前端界面中，但可以传递到后台`Controller`。

  ```html
  <input type="hidden" name="id" th:value="${emp.getId()}">
  ```

#### 员工信息修改后端

通过前端`update.html`修改的员工信息传递到后台`Controller`进行提交，代码如下：

```java
@PostMapping("/updateEmp")
public String updateEmp(Employee employee) {
    System.out.println(employee.getId());
    employeeDao.save(employee);
    return "redirect:/emps";
}
```

注意：

- `employeeDao.save(employee)`保存修改后的员工信息，如果在`update.html`中不使用隐藏域`type="hidden"`来获取员工的`id`信息，后端代码则获取不到员工的`id`信息，此时`save()`方法会创建出一个新的员工信息（由于`id`自增）而并非是修改原有员工信息

### 删除员工信息

- `list.html`删除按钮代码

  ```html
  <a type="button" class="btn btn-sm btn-danger" th:href="@{/delemp/}+${emp.getId()}">删除</a>
  ```

- 后端Controller接收参数删除员工信息

  ```java
  @GetMapping("/delemp/{id}")
  public String deleteEmp(@PathVariable("id") Integer id) {
  	employeeDao.delete(id);
  	return "redirect:/emps";
  }
  ```

### 404 页面

`SpringBoot`的 404 页面只需要在`templates`文件夹下创建一个`error`文件夹，在`error`文件夹下创建一个`404.html`文件即可

![image-20211210164449574](SpringBoot 基础.assets/image-20211210164449574.png)

![image-20211210164528842](SpringBoot 基础.assets/image-20211210164528842.png)

当输入不存在或者错误的`URL`时候便会跳转到 404 错误页面

### 退出功能

点击`退出`按钮，退回到`登录`界面

![image-20211210164826992](SpringBoot 基础.assets/image-20211210164826992.png)

- 在`commons.html`中修改`退出`按钮代码

  ```html
  <a class="nav-link" th:href="@{/user/logout}">退出</a>
  ```

- 在`LoginController.java`中编写注销页面代码

  ```java
  @RequestMapping("/user/logout")
  public String logout(HttpSession session) {
      session.invalidate();
      return "redirect:/index.html";
  }
  ```

# SpringBoot 整合数据库

- 对于数据访问层，无论是 `SQL`(关系型数据库) 还是 `NOSQL`(非关系型数据库)，`Spring Boot` 底层都是采用 `Spring Data` 的方式进行统一处理
- Spring Boot 底层都是采用 Spring Data [Sping Data 官网](https://gitee.com/link?target=https%3A%2F%2Fspring.io%2Fprojects%2Fspring-data)的方式进行统一处理各种数据库
- 数据库相关的启动器 ：可以参考[官方文档](https://gitee.com/link?target=https%3A%2F%2Fdocs.spring.io%2Fspring-boot%2Fdocs%2F2.2.5.RELEASE%2Freference%2Fhtmlsingle%2F%23using-boot-starter)

## 整合 JDBC

### SQL 模块引入

新建项目 `springboot-jdbc-study` ，需要引入`SQL` 模块：

![image-20211213095112962](SpringBoot 基础.assets/image-20211213095112962.png)

项目建好之后，在`pom`文件会自动为我们导入了如下的依赖包：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### yaml 配置文件连接数据库

项目建好之后，使用`yaml` 配置文件连接数据库

```yaml
spring:
  datasource:
    username: root
    password: 1234
    # ?serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/tmall?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
```

测试代码：

```java
@SpringBootTest
class DemoApplicationTests {
    
    @Autowired
    DataSource dataSource;
    
    @Test
    void contextLoads() throws SQLException {
        //看一下默认数据源
        System.out.println(dataSource.getClass());
        //获得连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        //关闭连接
        connection.close();
    }
}
```

结果：可以看到默认给项目配置的数据源为 : `class com.zaxxer.hikari.HikariDataSource` 

### JDBCTemplate

有了数据源`com.zaxxer.hikari.HikariDataSource`，便可以拿到数据库连接`java.sql.Connection`，之后就可以使用原生的 `JDBC` 语句来操作数据库

**JdbcTemplate主要提供以下几类方法：**

- `execute`方法：可以用于执行任何`SQL`语句，一般用于执行`DDL`语句
- `update`方法及`batchUpdate`方法：`update`方法用于执行新增、修改、删除等语句；`batchUpdate`方法用于执行批处理相关语句
- `query`方法及`queryForXXX`方法：用于执行查询相关语句
- `call`方法：用于执行存储过程、函数相关语句

编写一个`Controller`，注入 `jdbcTemplate`，编写测试方法进行访问测试：

```java
@RestController
public class JDBCController {

    @Autowired
    JdbcTemplate jdbcTemplate;

    // 查询数据库表 user 的所有信息
    // 直接获取数据库的东西—Map
    @GetMapping("/userList")
    public List<Map<String,Object>> userList() {
        String sql = "select * from usertest";
        List<Map<String,Object>> maps = jdbcTemplate.queryForList(sql);
        return maps;
    }

    @GetMapping("/addAddress")
    public String addUser() {
        String sql = "insert into tmall.usertest(id, username, pwd) values(3, 'Alixs', 666789)";
        jdbcTemplate.update(sql);
        return "update-ok";
    }

    @GetMapping("/updateUser/{id}")
    public String updateUser(@PathVariable("id") int id) {
        String sql = "update tmall.usertest set username  = ?,pwd = ? where id = " + id;
        //封装
        Object[] objects = new Object[2];

        objects[0] = "Maryland";
        objects[1] = 444789;

        jdbcTemplate.update(sql,objects);
        return "update-ok";
    }

    @GetMapping("/deleteUser/{id}")
    public String deleteUser(@PathVariable("id") int id) {
        String sql = "delete from tmall.usertest where id = ?";
        jdbcTemplate.update(sql,id);
        return "update-ok";
    }
}
```

数据库表`usertest`如下：

```sql
DROP TABLE IF EXISTS `usertest`;
CREATE TABLE `usertest` (
  `id` char(6) CHARACTER  NOT NULL,
  `username` varchar(50) CHARACTER  NOT NULL,
  `pwd` char(6) CHARACTER  NOT NULL,
  PRIMARY KEY (`id`) 
) ;

-- ----------------------------
-- Records of usertest
-- ----------------------------
INSERT INTO `usertest` VALUES ('1', 'Maryland', '444789');
INSERT INTO `usertest` VALUES ('3', 'Alixs', '666789');

SET FOREIGN_KEY_CHECKS = 1;
```

## 集成 Druid

- Java程序很大一部分要操作数据库，为了提高性能操作数据库的时候，又不得不使用数据库连接池
- Druid 是阿里巴巴开源平台上一个数据库连接池实现，结合了 C3P0、DBCP 等 DB 池的优点，同时加入了日志监控
- `com.alibaba.druid.pool.DruidDataSource` 基本配置参数如下：

|           **配置**            |     **缺省值**     | **说明**                                                     |
| :---------------------------: | :----------------: | :----------------------------------------------------------- |
|             name              |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。 如果没有配置，将会生成一个名字，格式是：“DataSource-” + System.identityHashCode(this) |
|            jdbcUrl            |                    | 连接数据库的url，不同数据库不一样。例如： mysql : jdbc:mysql://10.20.153.104:3306/druid2 oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
|           username            |                    | 连接数据库的用户名                                           |
|           password            |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：[https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter](https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Falibaba%2Fdruid%2Fwiki%2F%E4%BD%BF%E7%94%A8ConfigFilter) |
|        driverClassName        |  根据url自动识别   | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下) |
|          initialSize          |         0          | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
|           maxActive           |         8          | 最大连接池数量                                               |
|            maxIdle            |         8          | 已经不再使用，配置了也没效果                                 |
|            minIdle            |                    | 最小连接池数量                                               |
|            maxWait            |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
|    poolPreparedStatements     |       false        | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
|   maxOpenPreparedStatements   |         -1         | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
|        validationQuery        |                    | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。 |
|    validationQueryTimeout     |                    | 单位:秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法 |
|         testOnBorrow          |        true        | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
|         testOnReturn          |       false        | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
|         testWhileIdle         |       false        | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效 |
| timeBetweenEvictionRunsMillis |  1分钟 ( 1.0.14 )  | 有两个含义： 1) Destroy线程会检测连接的间隔时间 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
|    numTestsPerEvictionRun     |                    | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
|  minEvictableIdleTimeMillis   | 30分钟 ( 1.0.14 )  | 连接保持空闲而不被驱逐的最长时间                             |
|      connectionInitSqls       |                    | 物理连接初始化的时候执行的sql                                |
|        exceptionSorter        | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
|            filters            |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall |
|         proxyFilters          |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |

### 配置 Druid 数据源

- `Druid` 数据源的使用需要在`pom`文件引入依赖

  ```xml
  <!--druid数据源-->
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.23</version>
  </dependency>
  ```

- 之后在`yaml`文件中指定使用该数据源

  ```yaml
  spring:
    datasource:
      username: root
      password: 1234
      # ?serverTimezone=UTC解决时区的报错
      url: jdbc:mysql://localhost:3306/tmall?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
      driver-class-name: com.mysql.cj.jdbc.Driver
      # 指定数据源
      type: com.alibaba.druid.pool.DruidDataSource
  ```

- 配置好数据源进行测试，数据源发生改变，如下图所示：

![image-20211214170658597](SpringBoot 基础.assets/image-20211214170658597.png)

- 可以在`yaml`文件中设置数据源连接初始化大小、最大连接数、等待时间、最小连接数等设置项

  ```yaml
  # Druid 数据源专有配置
  initialSize: 5
  minIdle: 5
  maxActive: 20
  maxWait: 60000
  timeBetweenEvictionRunsMillis: 60000
  minEvictableIdleTimeMillis: 300000
  validationQuery: SELECT 1 FROM DUAL
  testWhileIdle: true
  testOnBorrow: false
  testOnReturn: false
  poolPreparedStatements: true
  
  # 配置监控统计拦截的 filters, stat:监控统计、log4j: 日志记录、wall: 防御 sql 注入
  # 如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
  # 则导入 log4j 依赖即可, Maven 地址 : https://mvnrepository.com/artifact/log4j/log4j
  filters: stat,wall,log4j
  maxPoolPreparedStatementPerConnectionSize: 20
  useGlobalDataSourceStat: true
  connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
  ```

- `yaml`文件中配置`log4j`日志记录之后需要在`pom`文件中引入依赖：

  ```xml
  <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
  </dependency>
  ```

- `yaml`文件中配置好`Druid`的全局参数之后，需要手动绑定为 `DruidDataSource` 参数并添加到容器中，创建`config`文件夹并创建类`DruidConfig`

  ```java
  @Configuration
  public class DruidConfig {
      @ConfigurationProperties(prefix = "spring.datasource")
      @Bean
      public DataSource druidDataSource() {
          return new DruidDataSource();
      }
  }
  ```

  `@ConfigurationProperties`将`yaml`文件中配置的数据源进行绑定

### 配置 Druid 数据源监控

`Druid` 数据源具有监控的功能，并提供了一个 `web` 界面方便用户查看，需要设置 Druid 的后台管理页面，比如登录账号、密码等，在类`DruidConfig`添加源码如下

```java
//配置 Druid 监控管理后台的 Servlet
//内置 Servlet 容器时没有 web.xml 文件，所以使用 Spring Boot 的注册 Servlet 方式
@Bean
public ServletRegistrationBean statViewServlet() {
    ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");

    // 这些参数可以在 com.alibaba.druid.support.http.StatViewServlet
    // 的父类 com.alibaba.druid.support.http.ResourceServlet 中找到
    Map<String, String> initParams = new HashMap<>();
    initParams.put("loginUsername", "root"); //后台管理界面的登录账号
    initParams.put("loginPassword", "admin"); //后台管理界面的登录密码

    //后台允许谁可以访问
    //initParams.put("allow", "localhost")：表示只有本机可以访问
    //initParams.put("allow", "")：为空或者为null时，表示允许所有访问
    initParams.put("allow", "");
    //deny: Druid 后台拒绝谁访问
    //initParams.put("kuangshen", "192.168.1.20");表示禁止此ip访问

    //设置初始化参数
    bean.setInitParameters(initParams);
    return bean;
}
```

注意以上代码中`loginUsername`、`loginPassword`是`Druid`中固定参数，自己不可以随意更改

配置完毕后，我们可以选择访问：http://localhost:8080/druid/login.html ，出现以下界面

![image-20211214215345159](SpringBoot 基础.assets/image-20211214215345159.png)

输入代码配置好的用户名和密码，进入页面

![image-20211214215444358](SpringBoot 基础.assets/image-20211214215444358.png)

测试输入http://localhost:8080/userList ，可以看到监控信息如下：

![image-20211214215648118](SpringBoot 基础.assets/image-20211214215648118.png)

### 配置 Druid web 监控 filter 过滤器

在类`DruidConfig`添加源码如下：

```java
//配置 Druid 监控 之  web 监控的 filter
//WebStatFilter : 用于配置Web和Druid数据源之间的管理关联监控统计
@Bean
public FilterRegistrationBean webStatFilter() {
    FilterRegistrationBean bean = new FilterRegistrationBean();
    bean.setFilter(new WebStatFilter());

    //exclusions：设置哪些请求进行过滤排除掉，从而不进行统计
    Map<String, String> initParams = new HashMap<>();
    initParams.put("exclusions", "*.js,*.css,/druid/*,/jdbc/*");
    bean.setInitParameters(initParams);

    //"/*" 表示过滤所有请求
    bean.setUrlPatterns(Arrays.asList("/*"));
    return bean;
}
```

## 整合Mybatis

`Mybatis`官方文档：http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

创建项目`springboot-mybatis-study`，`SpringBoot`整合`Mybatis`需要以下步骤

1. `pom`文件中导入 `MyBatis` 所需要的依赖

   ```xml
   <!--Mybatis-->
   <dependency>
   	<groupId>org.mybatis.spring.boot</groupId>
   	<artifactId>mybatis-spring-boot-starter</artifactId>
   	<version>2.2.0</version>
   </dependency>
   ```

2. 配置数据库连接池，可以集成 `Druid`，建议配置好之后先进行测试，测试数据库是否连接成功

3. 创建实体类User，建议使用`Lombok`

   ```java
   @Data //Lombok标签
   @AllArgsConstructor
   @NoArgsConstructor
   public class User {
       //属性名称要与数据库对应表字段名称一致(不区分大小写)
       private int id;
       private String Username;
       private int pwd;
   }
   ```

4. 创建实体类`User`对应的`Mapper`接口`UserMapper`，在该接口中定义`User`类的相关方法

   ```java
   @Mapper
   @Repository
   public interface UserMapper {
       List<User> queryUserList();
   
       User queryUserById(Integer id);
   
       int addUser(User User);
   
       int updateUser(User user);
   
       int deleteUser(int id);
   }
   ```

5. 创建接口`UserMapper`对应的映射文件`UserMapper.xml`，建议统一将映射文件放在`resources/mybatis/mapper`路径下

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <mapper namespace="com.springboot.mybatis.springbootmybatis.mapper.UserMapper">
       <select id="queryUserList" resultType="User">
           select * from tmall.usertest;
       </select>
   
       <select id="queryUserById" resultType="User">
           select * from tmall.usertest where id = #{id}
       </select>
   
       <insert id="addUser" parameterType="User">
           insert into tmall.usertest(id, username, pwd) values (#{id}, #{username}, #{pwd});
       </insert>
   
       <update id="updateUser" parameterType="User">
           update tmall.usertest set username=#{username},pwd = #{pwd} where id = #{id};
       </update>
   
       <delete id="deleteUser" parameterType="int">
           delete from tmall.usertest where id = #{id}
       </delete>
   </mapper>
   ```

6. 在`yaml`文件中整合`Mybatis`，将映射文件和实体类对应起来

   ```yaml
   # 整合 Mybatis
   mybatis:
     type-aliases-package: com.springboot.mybatis.springbootmybatis.pojo
     mapper-locations: classpath:mybatis/mapper/*.xml
   ```

7. 编写相对应的`Controller`，创建`UserController`类

   ```java
   @RestController
   public class UserController {
   
       @Autowired
       private UserMapper userMapper;
   
       // 查询员工列表所有信息
       @GetMapping("/queryUserList")
       public List<User> queryUserList() {
           List<User> userList = userMapper.queryUserList();
           for (User user : userList){
               System.out.println(user);
           }
           return userList;
       }
   
       // 通过 id 查询对应员工信息
       @GetMapping("/queryUserById/{id}")
       public User queryUserById(@PathVariable("id") Integer id) {
           User user = userMapper.queryUserById(id);
           return user;
       }
   
       // 添加一个用户
       @GetMapping("/addUser")
       public String addUser() {
           userMapper.addUser(new User(7,"Jack",123456));
           return "ok";
       }
   
       // 修改一个用户
       @GetMapping("/updateUser")
       public String updateUser() {
           userMapper.updateUser(new User(7,"JackSon",111456));
           return "ok";
       }
   
       // 删除一个 User
       @GetMapping("/deleteUser")
       public String deleteUser() {
           userMapper.deleteUser(7);
           return "ok";
       }
   }
   ```

项目最终结构图如下所示：

![image-20211215161718329](SpringBoot 基础.assets/image-20211215161718329.png)

## 整合Redis

### 整合方式

`SpringBoot`整合`Redis`步骤如下：

1. `pom`文件中导入`xml`文件

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
       <version>2.6.1</version>
   </dependency>
   ```

2. 编写配置文件内容

   ```yaml
   spring:
     redis:
       host: 101.43.164.126
       port: 6379
       # password:
       jedis:
         pool:
           max-active: 8    # 控制一个 pool 可分配多少个 Jedis 实例
           max-wait: -1ms   # 获取一个 pool 中 Jedis 最大的等待时间
           max-idle: 500    # pool 中最大的空闲连接数
           min-idle: 0      # pool 中保持最小的空闲可用连接数, 这部分不被回收
       lettuce:
         shutdown-timeout: 0ms
   ```

3. 使用`RedisTemplate`

   ```java
   @SpringBootTest
   class Redis02SpringbootApplicationTests {
   
       @Autowired
       private RedisTemplate redisTemplate;
   
       @Test
       void contextLoads() {
           // redisTemplate 操作不同的数据类型
           // opsForValue 操作字符串,类似 String
           // opsForList 操作 List,类似 List
           // opsForHash 操作 Hash
   
           // 除了基本的操作,常用方法都可以直接通过redisTemplate操作,如事务和基本的CRUD
   
           // 获取连接对象
           //RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
           //connection.flushDb();
           //connection.flushAll();
   
           redisTemplate.opsForValue().set("mykey","wyj");
           System.out.println(redisTemplate.opsForValue().get("mykey"));
       }
   }
   ```

4. 测试结果：

   测试结果可以连接`Redis`并输出相应内容

### 定制RedisTemplate

当`Redis`中存`Java`对象时，如创建一个`User`实体类：

```java
@Component
public class User {
    private String username;
    private String password;
}

User user = new User();
```

实际项目开发中`redisTemplate`不会直接存储实体类对象，例如以下代码：

```java
User user = new User("wyj", "123456");
redisTemplate.opsForValue().set("key1",user);
System.out.println(redisTemplate.opsForValue().get("key1"));
```

程序运行会报错：

![image-20220103211816970](SpringBoot 基础.assets/image-20220103211816970.png)

错误提示`User`对象需要序列化，可以自定义`RedisTemplate`实现序列化，代码如下：

创建`Redis`配置类：

```java
@Configuration
@SuppressWarnings("all")  //抑制所有类型的警告
public class RedisConfig {

    /*RedisAutoConfiguration
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }*/
    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {

        RedisTemplate<String,Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);

        // 序列化配置
        // 使用 Jackson 序列化任意对象类型数据
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        // String 序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key 采用 String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash 的 key 也采用 String 的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value 序列化方式采用 jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash 的 value 序列化方式采用 jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();

        return template;
    }

}
```

### 自定义Redis工具类

使用`RedisTemplate`需要频繁调用`.opForxxx`才能执行对应的操作，该方式代码效率低下，实际开发可以将常用的公共`API`抽取出来封装成为一个工具类，然后直接使用工具类来间接操作`Redis`

参考网址：<https://www.cnblogs.com/zeng1994/p/03303c805731afc9aa9c60dbbd32a323.html>

代码如下：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Component
public final class RedisUtil {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================common============================
    /**
     * 指定缓存失效时间
     * @param key 键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete((Collection<String>) CollectionUtils.arrayToList(key));
            }
        }
    }

    // ============================String=============================
    /**
     * 普通缓存获取
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     * @param key 键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     * @param key 键
     * @param value 值
     * @param time 时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     * @param key 键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     * @param key 键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    // ================================Map=================================
    /**
     * HashGet
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     * @param key 键
     * @param map 对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     * @param key 键
     * @param item 项
     * @param value 值
     * @param time 时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     * @param key 键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     * @param key 键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     * @param key 键
     * @param item 项
     * @param by 要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     * @param key 键
     * @param item 项
     * @param by 要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    // ============================set=============================
    /**
     * 根据key获取Set中的所有值
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     * @param key 键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     * @param key 键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     * @param key 键
     * @param time 时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     * @param key 键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    // ===============================list=================================

    /**
     * 获取list缓存的内容
     * @param key 键
     * @param start 开始
     * @param end 结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     * @param key 键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key 键
     * @param value 值
     * @param time 时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     * @param key 键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     * @param key 键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
}
```

### 部分源码讲解

1. `lettuce`和`Jedis`的定位都是`Redis`的客户端实现，可以直接连接`Redis`的服务端。`springboot 2.x`后 ，z `Jedis` 被 `lettuce` 替换。

   > `Jedis`：采用的直连，多个线程操作是不安全的，需要使用`Jedis pool`连接池保证其安全，类似于`BIO`模式
   >
   > `lettuce`：采用`netty`方式连接，实例可以在多个线程中共享，不存在线程不安全的情况，类似于`NIO`模式

2. `SpringBoot`整合一个组件时，一般都会有一个自动配置类`XxxAutoConfiguration`，并且对应一个配置属性类`XxxProperties`，`RedisAutoConfiguration`类部分源码如下：

   ![image-20220103204352992](SpringBoot 基础.assets/image-20220103204352992.png)

   `RedisAutoConfiguration`类中存在两个Bean对象：

   - `RedisTemplate``
   - ``StringRedisTemplate`

   通过`RedisTemplate`和`StringRedisTemplate`对象可以分别操作`Redis`和`Redis`中的`String`数据类型。在`RedisProperties`中，说明了配置文件需要编写的内容：

   ![image-20220103204736745](SpringBoot 基础.assets/image-20220103204736745.png)

3. `RedisTemplate`类内部实现了对实体类对象的序列化方式：

   ![image-20220103214413587](SpringBoot 基础.assets/image-20220103214413587.png)

   默认的序列化方式为`JDK`序列器：

   ![image-20220103214812245](SpringBoot 基础.assets/image-20220103214812245.png)

   `RedisSerializer`提供了多种序列化方案，可以在自定义RedisTemplate中设置：

   ![image-20220103215027555](SpringBoot 基础.assets/image-20220103215027555.png)

# SpringSecurity入门

## SpringSecurity环境搭建

1、新建`SpringBoot`项目`SpringSecurity-first-study`，选中`Spring Security`选项

![image-20211217174539328](SpringBoot 基础.assets/image-20211217174539328.png)

`pom`文件会自动引入依赖包：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

2、导入静态资源文件，路径如下：

![image-20211221094343334](SpringBoot 基础.assets/image-20211221094343334.png)

3、编写`Controller`类`RouterController`，测试代码

```java
@Controller
public class RouterController {

    @RequestMapping({"/","/index"})
    public String index(){
        return "index";
    }

    @RequestMapping("/level1/{id}")
    public String level1(@PathVariable("id") int id){
        return "views/level1/"+id;
    }

    @RequestMapping("/level2/{id}")
    public String level2(@PathVariable("id") int id){
        return "views/level2/"+id;
    }

    @RequestMapping("/level3/{id}")
    public String level3(@PathVariable("id") int id){
        return "views/level3/"+id;
    }
}
```

4、测试结果：

测试发现`SpringBoot`项目启动成功，访问任意URL路径都会跳转到一个登录页面：

![image-20211221094745654](SpringBoot 基础.assets/image-20211221094745654.png)

`SpringBoot`项目中使用了`SpringSecurity`，默认的`SpringSecurity`便生效了，此时的接口都是被保护的，我们需要通过验证通过才能正常的访问。`SpringSecurity`提供了一个默认的用户，用户名是user，而密码则是启动项目的时候自动生成的

![image-20211221095224062](SpringBoot 基础.assets/image-20211221095224062.png)

如果不需要验证访问，可以在项目中添加一个配置类，配置不需要登录验证功能，代码如下：

```java
@Configuration
@EnableWebSecurity
public class CloseSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        //配置不需要登陆验证
        http.authorizeRequests().anyRequest().permitAll().and().logout().permitAll();
    }
}
```

最终测试成功：
![image-20211221101212513](SpringBoot 基础.assets/image-20211221101212513.png)

## 初识SpringSecurity

`SpringSecurity`是针对`Spring`项目的安全框架，也是`SpringBoot`底层安全模块默认的技术选型，他可以实现强大的`Web`安全控制，对于安全控制，我们仅需要引入`spring-boot-starter-security`模块，进行少量的配置，即可实现强大的安全管理功能。

`SpringSecurity`中几个重要的类：

- `WebSecurityConfigurerAdapter`：自定义`Security`安全策略

- `AuthenticationManagerBuilder`：自定义认证策略

- `@EnableWebSecurity`：开启`WebSecurity`模式

- 类`WebSecurityConfigurerAdapter`中重要的方法：

  ```java
  protected void configure(HttpSecurity http) throws Exception {
      http.authorizeRequests().antMatchers(&quot;/**&quot;).hasRole(&quot;USER&quot;).and().formLogin()
  	  				.usernameParameter(&quot;username&quot;) // default is username
  	  				.passwordParameter(&quot;password&quot;) // default is password
  	  				.loginPage(&quot;/authentication/login&quot;) // default is /login with an HTTP get
  	  				.failureUrl(&quot;/authentication/login?failed&quot;) // default is /login?error
  	  				.loginProcessingUrl(&quot;/authentication/login/process&quot;); // default is /login
  	  																		// with an HTTP
  	  																		// post
  }
  ```

`SpringSecurity`的两个主要目标是 “认证” 和 “授权”（访问控制）

- “认证”（`Authentication`）

  身份验证是关于验证身份的凭据，如用户名、用户`ID`和密码，以验证身份权限，身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用

- “授权” （`Authorization`）

  授权发生在系统成功验证用户的身份后，最终会授予用户访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限

## 认证和授权

如果不同的用户登录可以访问不同的`Level`下的标签，如下图所示：

![image-20211221162851072](SpringBoot 基础.assets/image-20211221162851072.png)

可以为每一个`Level`设置一个权限，并对每一个权限下绑定不同的用户，此时需要使用`SpringSecurity`中的认证和授权功能。

 使用`SpringSecurity`环境搭建的项目进行认证和授权功能的实现，实现流程如下：

1. 引入`SpringSecurity`模块

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 编写`SpringSecurity`基础配置类

   参考官网：https://spring.io/projects/spring-security 

   ```java
   import org.springframework.security.config.annotation.web.builders.HttpSecurity;
   import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
   import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
   
   @EnableWebSecurity // 开启 WebSecurity 模式
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
   
       }
   }
   ```

   ![image-20211221104359440](SpringBoot 基础.assets/image-20211221104359440.png)

   此时需要将`CloseSecurityConfig`类中不需要登陆验证的代码也放在上述代码的`configure()`方法中，否则测试会报错

3. 在`configure()`方法中定制请求的授权规则

   ```java
   @Override
   protected void configure(HttpSecurity http) throws Exception {
        
       http.csrf().disable();
       //配置不需要登陆验证
       http.authorizeRequests().anyRequest().permitAll().and().logout().permitAll();
       
       http.authorizeHttpRequests().antMatchers("/").permitAll() // 首页所有人可以访问
               .antMatchers("/level1/**").hasRole("vip1") // 拥有 vip1 权限的角色可以访问 level1 下的页面
               .antMatchers("/level2/**").hasRole("vip2") // 拥有 vip2 权限的角色可以访问 level2 下的页面
               .antMatchers("/level3/**").hasRole("vip3");// 拥有 vip3 权限的角色可以访问 level3 下的页面
   }
   ```

4. 测试

   ![image-20211221110706100](SpringBoot 基础.assets/image-20211221110706100.png)

   进入首页点击`Level-1-1`等标签会发现出现以下`403`错误

   ![image-20211221110756106](SpringBoot 基础.assets/image-20211221110756106.png)

   以上原因是因为我们没有设置登录的角色，请求执行时需要登录的角色拥有对应的权限才可以，例如角色`root`想访问`level1`页面下的资源文件，必须要拥有对应的访问权限`vip1`

5. 在`configure()`方法中加入以下配置，开启自动配置的登录功能

   ```java
   // 开启自动配置的登录功能
   // /login 请求来到登录页
   // /login?error 重定向到这里表示登录失败
   http.formLogin();
   ```

6. 再次测试，发现没有权限的时候，会跳转到登录的页面

   ![image-20211221112728095](SpringBoot 基础.assets/image-20211221112728095.png)

7. 跳转到登录页面之后，可以定义认证规则，重写`configure(AuthenticationManagerBuilder auth)`方法

   ```java
   //定义认证规则
   @Override
   protected void configure(AuthenticationManagerBuilder auth) throws Exception {
   
       auth.inMemoryAuthentication()
                   .withUser("wyj").password("123456").roles("vip2","vip3")//用户 wyj 具有权限 vip2 和 vip3
                   .and()
                   .withUser("root").password("123456").roles("vip1","vip2","vip3") 
                   .and()
                   .withUser("guest").password("123456").roles("vip1","vip2");    
   }
   ```
   
8. 再次测试，输入设置的`Username`和对应`Password`之后，发现会报错

   ![image-20211221113637778](SpringBoot 基础.assets/image-20211221113637778.png)

   ![image-20211221113708101](SpringBoot 基础.assets/image-20211221113708101.png)

   原因在于`SpringSecurity`要求要将前端传过来的密码进行某种方式加密，否则就无法登录，修改认证规则代码

   ```java
   //定义认证规则
   @Override
   protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       //Spring security 5.0中新增了多种加密方式，也改变了密码的格式。
       //要想我们的项目还能够正常登陆，需要修改一下configure中的代码。我们要将前端传过来的密码进行某种方式加密
       //spring security 官方推荐的是使用bcrypt加密方式。
   
       auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
               .withUser("wyj").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3")
               .and()
               .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2","vip3")
               .and()
               .withUser("guest").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1","vip2");
   
   }
   ```

9. 再次测试，发现登录成功，并且每个角色只能访问自己认证下的规则

## 自动注销

可以为首页添加一个注销功能，步骤如下：

1. 在`configure(HttpSecurity http)`方法中开启自动配置的注销的功能

   ```java
   //定制请求的授权规则
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       //....
       //开启自动配置的注销的功能
       // 开启自动配置的注销的功能 /logout 为注销请求
       http.logout();
   }
   ```

2. 在前端`index.html`导航栏中，增加一个注销的按钮，注意注销请求的`URL`是`/logout`

   ```html
   <a class="item" th:href="@{/logout}">
       <i class="address card icon"></i> 注销
   </a>
   ```

3. 点击`Level-2-1`进入`Level2`界面之后，在`Level2`界面点击注销直接退到登录界面，如果要跳转到首页，需要在`configure(HttpSecurity http)`方法中修改`http.logout()`方法

   ![image-20211221164641589](SpringBoot 基础.assets/image-20211221164641589.png)

   ![image-20211221164610083](SpringBoot 基础.assets/image-20211221164610083.png)

   ```java
   //定制请求的授权规则
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       //....
   
       //开启自动配置的注销的功能
       // 开启自动配置的注销的功能 /logout 为注销请求
       //http.logout();
   
       // .logoutSuccessUrl("/"); 注销成功来到首页
       http.logout().logoutSuccessUrl("/");
   }
   ```

   之后，在点击注销按钮时，会退出到首页

   ![image-20211221165543382](SpringBoot 基础.assets/image-20211221165543382.png)

## 权限控制：整合Thymeleaf

如果用户`guest`只可以访问`Level1`和`Level2`，不希望用户`guest`登录之后看到`Level3`，并且用户登录之后隐藏登录按钮，用户注销之后隐藏注销按钮，如下所示

![image-20211221172153275](SpringBoot 基础.assets/image-20211221172153275.png)

以上需求需要结合`thymeleaf`中的功能，流程如下：

1. 引入`Maven`依赖

   ```xml
   <dependency>
   	<groupId>org.thymeleaf.extras</groupId>
   	<artifactId>thymeleaf-extras-springsecurity5</artifactId>
   </dependency>
   ```

2. 首页`index.html`导入命名空间

   ```html
   xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5
   ```

3. 首页`index.html`修改导航栏，增加认证判断，进行测试

   ```html
   <!--登录注销-->
   <div class="right menu">
       <!--未登录, 显示登录按钮-->
       <div sec:authorize="!isAuthenticated()">
           <a class="item" th:href="@{/login}">
               <i class="address card icon"></i> 登录
           </a>
       </div>
   
       <!--已登录, 界面显示登陆的用户名-->
       <div sec:authorize="isAuthenticated()">
           <a class="item">
               <i class="address card icon"></i>
               <!--用户名：--><span sec:authentication="principal.username"></span>
               <!--角色：<span sec:authentication="principal.authorities"></span>-->
           </a>
       </div>
   
       <!--已登录, 显示注销按钮-->
       <div sec:authorize="isAuthenticated()">
           <a class="item" th:href="@{/logout}">
               <i class="address card icon"></i> 注销
           </a>
       </div>
   </div>
   ```

   之后使用`URL`访问`http://localhost:8080/login` 登录请求，登录用户名为`guest`，登录成功后发现登录按钮消失，并显示出了登录的用户名，页面如下：

   ![image-20211221180704166](SpringBoot 基础.assets/image-20211221180704166.png)
   
   点击注销，会显示如下界面：
   
   ![image-20211222093338700](SpringBoot 基础.assets/image-20211222093338700.png)
   
   点击`Log Out`退出之后，显示首页，注销按钮消失
   
   ![image-20211222093550604](SpringBoot 基础.assets/image-20211222093550604.png)

4. 如果点击注销按钮，出现`404`页面，是因为它默认防止`csrf`跨站请求伪造，因为会产生安全问题，可以将请求改为`post`表单提交，或者在`SpringSecurity`中关闭`csrf`功能，在`configure(HttpSecurity http)`方法中添加以下代码：

   ```java
   // 关闭 csrf 功能: 跨站请求伪造, 默认只能通过 post 方式提交 logout 请求
   http.csrf().disable();
   ```

5. 继续修改`index.html`内容，完善角色功能块权限控制，进行测试

   ```html
   <div class="column" >
       <div class="ui raised segment">
           <!-- sec:authorize="hasRole('vip1')" -->
           <div class="ui" sec:authorize="hasRole('vip1')">
               <div class="content">
                   <h5 class="content">Level 1</h5>
                   <hr>
                   <div><a th:href="@{/level1/1}"><i class="bullhorn icon"></i> Level-1-1</a></div>
                   <div><a th:href="@{/level1/2}"><i class="bullhorn icon"></i> Level-1-2</a></div>
                   <div><a th:href="@{/level1/3}"><i class="bullhorn icon"></i> Level-1-3</a></div>
               </div>
           </div>
       </div>
   </div>
   
   <div class="column">
       <div class="ui raised segment">
           <div class="ui" sec:authorize="hasRole('vip2')">
               <div class="content" >
                   <h5 class="content">Level 2</h5>
                   <hr>
                   <div><a th:href="@{/level2/1}"><i class="bullhorn icon"></i> Level-2-1</a></div>
                   <div><a th:href="@{/level2/2}"><i class="bullhorn icon"></i> Level-2-2</a></div>
                   <div><a th:href="@{/level2/3}"><i class="bullhorn icon"></i> Level-2-3</a></div>
               </div>
           </div>
       </div>
   </div>
   
   <div class="column">
       <div class="ui raised segment">
           <div class="ui" sec:authorize="hasRole('vip3')">
               <div class="content">
                   <h5 class="content">Level 3</h5>
                   <hr>
                   <div><a th:href="@{/level3/1}"><i class="bullhorn icon"></i> Level-3-1</a></div>
                   <div><a th:href="@{/level3/2}"><i class="bullhorn icon"></i> Level-3-2</a></div>
                   <div><a th:href="@{/level3/3}"><i class="bullhorn icon"></i> Level-3-3</a></div>
               </div>
           </div>
       </div>
   </div>
   ```

   使用`sec:authorize="hasRole('vip1')"`来控制界面需要显示内容

   进行测试，使用`guest`登录，发现只显示`Level1`和`Level2`

   ![image-20211222095843139](SpringBoot 基础.assets/image-20211222095843139.png)

## 记住我

如果要实现登录时记住密码的功能，登录之后，关闭浏览器，再次登录不需要输入用户名密码便可以直接登录成功，需要开启`记住我`功能。

**原理：**登录成功后，将`cookie`发送给浏览器保存，再次登录会带上该`cookie`信息，通过检查便可以免登录了；如果点击注销，则会删除该`cookie`信息

```java
//定制请求的授权规则
@Override
protected void configure(HttpSecurity http) throws Exception {
    // ...
    //记住我
    http.rememberMe();
}
```

选中`Remember me on this computer`，启动项目测试，发现登录界面出现以下内容：

![image-20211222102717480](SpringBoot 基础.assets/image-20211222102717480.png)

登录成功之后，浏览器保存了`Cookies`信息（记住我功能的实现原理：保存登录`Cookie`信息）：

![image-20211222103156331](SpringBoot 基础.assets/image-20211222103156331.png)

点击注销，`SpringSecurity`会自动删除该`Cookie`信息

## 定制登录页

以上代码中看到的登录页面都是`SpringSecurity`默认提供的登录页面，用户也可以自定义自己的登录界面，步骤如下：

1. 类`RouterController`中写登录`Controller`

   ```java
   @RequestMapping("/toLogin")
   public String toLogin() {
       return "views/login";
   }
   ```

2. `configure(HttpSecurity http)`方法中自定义登录页

   ```java
   //http.formLogin();
   //自定义登录页面
   http.formLogin().loginPage("/toLogin");
   ```

3. 前端首页`index.html`跳转到登录页面的路由指向自己定义的登录`URL`

   ```html
   <!--未登录, 显示登录按钮-->
   <div sec:authorize="!isAuthenticated()">
       <a class="item" th:href="@{/toLogin}">
           <i class="address card icon"></i> 登录
       </a>
   </div>
   ```

4. 之后进行测试，跳转到自定义的登录界面：

   ![image-20211222105222752](SpringBoot 基础.assets/image-20211222105222752.png)

5. 点击上图自定义登录界面的提交按钮，跳转到首页界面，需要修改`login.html`登录`form`表单的`th:action`

   ```html
   <form th:action="@{/toLogin}" method="post">
       <div class="field">
           <label>Username</label>
           <div class="ui left icon input">
               <input type="text" placeholder="Username" name="username">
               <i class="user icon"></i>
           </div>
       </div>
       ...
   </form>
   ```

   `th:action`中的`URL`和`configure(HttpSecurity http)`配置的自定义登录页面`URL`需一致，都为`/toLogin`。此时，登录成功，便可以自动跳转到首页。

   注意：在`th:action`中并没有配置跳转到首页`index.html`的`controller`，确可以完成跳转功能，也可以手动写一个`controller`实现跳转，`login.html`登录`form`表单的`th:action`如下：

   ```html
   <form th:action="@{/login}" method="post">
       <div class="field">
           <label>Username</label>
           <div class="ui left icon input">
               <input type="text" placeholder="Username" name="username">
               <i class="user icon"></i>
           </div>
       </div>
       ...
   </form>
   ```

   对应类`RouterController`添加`controller`如下：

   ```java
   @RequestMapping("/login")
   public String login() {
       return "redirect:/index.html";
   }
   ```

   也可以不写上面的`controller`， 在`configure(HttpSecurity http)`配置的自定义登录页面代码

   ```java
   // 自定义登录页面
   http.formLogin().loginPage("/toLogin").loginProcessingUrl("/login");
   ```

6. `login.html`登录`form`表单提交到后台的用户名和密码需要进行权限认证，`formLogin()`方法对用户名和密码进行验证，查看其代码，可知默认接收的用户名和密码参数名为`username`和`password`

   ```java
   http.authorizeRequests().antMatchers(&quot;/**&quot;).hasRole(&quot;USER&quot;).and().formLogin()
              .usernameParameter(&quot;username&quot;) // default is username
              .passwordParameter(&quot;password&quot;) // default is password
   ```

   如果对登录`form`表单用户名`name`属性和密码`name`属性进行修改：

   ```html
   <form th:action="@{/toLogin}" method="post">
       <div class="field">
           <label>Username</label>
           <div class="ui left icon input">
               <input type="text" placeholder="Username" name="uname">
               <i class="user icon"></i>
           </div>
       </div>
       <div class="field">
           <label>Password</label>
           <div class="ui left icon input">
               <input type="password" name="pwd">
               <i class="lock icon"></i>
           </div>
       </div>
       <input type="submit" class="ui blue submit button"/>
   </form>
   ```

   后台代码便接收不到提交的用户名和密码了，可以手动设置后台接收的用户名和参数名字：

   ```java
   // 自定义登录页面
   http.formLogin().loginPage("/toLogin").usernameParameter("uname").passwordParameter("pwd");
   ```

7. 在自定义登录页面再次实现`记住我`功能：

   在登陆页添加记住我的多选框：

   ```html
   <div class="field">
       <input type="checkbox" name="remember"> 记住我
   </div>
   ```

   后端`configure(HttpSecurity http)`验证处理

   ```java
   //定制记住我的参数
   http.rememberMe().rememberMeParameter("remember");
   ```

   `name`属性要和`rememberMeParameter()`参数一样，测试如下：

   ![image-20211222172833884](SpringBoot 基础.assets/image-20211222172833884.png)

# Shiro入门

## Shiro 简介

`Shiro`是一个功能强大和易于使用的`Java`安全框架，为开发人员提供一个直观而全面的解决方案的认证，授权，加密，会话管理

![image-20211223110049792](SpringBoot 基础.assets/image-20211223110049792.png)

**Shiro 四个主要的功能：**

- Authentication：身份认证/登录，验证用户是不是拥有相应的身份
- Authorization：授权，即权限验证，判断某个已经认证过的用户是否拥有某些权限访问某些资源，一般授权会有角色授权和权限授权
- Session Management：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的，web 环境中作用是和 HttpSession 是一样的
- Cryptography：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储

**Shiro 的其它几个特点：**

- Web Support：Web支持，可以非常容易的集成到Web环境；
- Caching：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；
- Concurrency：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；
- Testing：提供测试支持；
- Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；
- Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

![image-20211223110812447](SpringBoot 基础.assets/image-20211223110812447.png)

Shiro 的架构有 3 个主要概念：`Subject`， `SecurityManager`和`Realms`

- `Subject`：主体，相当于是请求过来的“用户”
- `SecurityManager`： 管理着所有 `Subject`，负责进行认证和授权、及会话、缓存的管理，是 `Shiro` 的心脏，所有具体的交互都通过 `SecurityManager` 进行拦截并控制
- `Realm`：一般我们都需要去实现自己的`Realm` ，可以有`1`个或多个 `Realm`，即当我们进行登录认证时所获取的安全数据来源(帐号/密码)

学习参考：https://www.cnblogs.com/progor/p/10970971.html

## SpringBoot整合Shiro环境搭建

步骤如下：

1. 创建`SpringBoot`项目`springboot-shiro-first`，导入`Shiro`依赖包

   ```xml
   <!--SpringBoot 整合 Shiro-->
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-spring</artifactId>
       <version>1.4.1</version>
   </dependency>
   ```

2. 编写简单的前端代码

   `index.html`：

   ```html
   <!DOCTYPE html >
   <html lang="en" xmlns:th="http://www.thymeleaf.org" >
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <h1>首页</h1>
       <p th:text="${msg}"></p>
       <a th:href="@{/user/add}">add</a> |
       <a th:href="@{/user/update}">update</a>
   </body>
   </html>
   ```

   `add.html`：

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <h1>add</h1>
   </body>
   </html>
   ```

   `update.html`:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <h1>update</h1>
   </body>
   </html>
   ```

3. 编写`controller`代码`MyController`：

   ```java
   @Controller
   public class MyController {
   
       @RequestMapping({"/","/index"})
       public String toIndex(Model model) {
           model.addAttribute("msg","hello, Shiro!");
           return "index";
       }
   
       @RequestMapping("/user/add")
       public String add() {
           return "user/add";
       }
   
       @RequestMapping("/user/update")
       public String update() {
           return "user/update";
       }
   }
   ```

4. 编写`Shiro`配置类`ShiroConfig`

   ```java
   package com.wyj.config;
   
   import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
   import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class ShiroConfig {
   
       // ShiroFilterFactoryBean
       @Bean
       public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityManager defaultWebSecurityManager) {
           ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
           // 设置安全管理器
           bean.setSecurityManager(defaultWebSecurityManager);
           return bean;
       }
   
       // DefaultWebSecurityManager
       @Bean(name = "securityManager")
       public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm) {
           DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
           // 关联 UserRealm
           securityManager.setRealm(userRealm);
           return securityManager;
       }
   
       // 创建 realm 对象, 需要自定义类
       // @Bean(name = "userRealm")
       @Bean
       public UserRealm userRealm() {
           return new UserRealm();
       }
   }
   ```

   注意：

   1. 以上代码为`Shiro`配置类的固定写法，需要创建`ShiroFilterFactoryBean`过滤器对象、`DefaultWebSecurityManager`对象、自定义`Realm`对象

   2. 通过`@Qualifier`标签拿到`userRealm()`方法创建的`Bean`对象，绑定到`getDefaultWebSecurityManager()`方法中的参数`userRealm`上

      ![image-20211224162145421](SpringBoot 基础.assets/image-20211224162145421.png)

5. 编写自定义的`Realm`类，需要继承`AuthorizingRealm`抽象类

   ```java
   /**
    * 自定义 UserRealm, 需要继承 AuthorizingRealm
    */
   public class UserRealm extends AuthorizingRealm {
   
       // 授权
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
           System.out.println("执行了=>授权doGetAuthorizationInfo");
           return null;
       }
   
       // 认证
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
           System.out.println("执行了=>认证doGetAuthenticationInfo");
           return null;
       }
   }
   ```

6. 测试搭建环境

   测试首页和`add`和`update`按钮正常跳转

   ![image-20211227100533969](SpringBoot 基础.assets/image-20211227100533969.png)

   ![image-20211227100555318](SpringBoot 基础.assets/image-20211227100555318.png)

   ![image-20211227100620027](SpringBoot 基础.assets/image-20211227100620027.png)

## 登录拦截

如果点击`add`、`update`时，要设置拦截功能：在进行页面跳转过程中首先会跳转到一个登录界面。`Shiro`可以通过拦截器链实现该功能，常见的拦截器有：

- `anon`：任何人都可以访问
- `authc`：只有认证后才可以访问
- `logout`：只有登录后才可以访问
- `roles`[角色名]：只有拥有特定角色才能访问
- `perms`["行为"]：只有拥有某种行为的才能访问

代码实现步骤如下：

1. 手动创建拦截跳转页面，并编写相应`controller`

   `login.html`：

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
       <title>登录</title>
       <!--semantic-ui-->
       <link href="https://cdn.bootcss.com/semantic-ui/2.4.1/semantic.min.css" rel="stylesheet">
   </head>
   <body>
   
   <!--主容器-->
   <div class="ui container">
   
       <div class="ui segment">
   
           <div style="text-align: center">
               <h1 class="header">登录</h1>
           </div>
   
           <div class="ui placeholder segment">
               <div class="ui column very relaxed stackable grid">
                   <div class="column">
                       <div class="ui form">
                           <form th:action="@{/toLogin}" method="post">
                               <div class="field">
                                   <label>Username</label>
                                   <div class="ui left icon input">
                                       <input type="text" placeholder="Username" name="uname">
                                       <i class="user icon"></i>
                                   </div>
                               </div>
                               <div class="field">
                                   <label>Password</label>
                                   <div class="ui left icon input">
                                       <input type="password" name="pwd">
                                       <i class="lock icon"></i>
                                   </div>
                               </div>
                               <input type="submit" class="ui blue submit button"/>
                           </form>
                       </div>
                   </div>
               </div>
           </div>
       </div>
   </div>
   
   <script th:src="@{/js/jquery-3.1.1.min.js}"></script>
   <script th:src="@{/js/semantic.min.js}"></script>
   
   </body>
   </html>
   ```

   `MyController.java`：

   ```java
   @RequestMapping("/toLogin")
   public String toLogin() {
       return "login";
   }
   ```

2. 编写登录拦截功能

   登录拦截功能代码的实现需要在`ShiroConfig`类中`getShiroFilterFactoryBean(...)`添加以下代码：

   ```java
   @Bean
   public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager")DefaultWebSecurityManager defaultWebSecurityManager) {
       // ...
   
       // (1)设置拦截器链 : anon authc logout roles perms
       Map<String, String> filterMap = new LinkedHashMap<>();
       //filterMap.put("/user/add","authc");
       //filterMap.put("/user/update","authc");
       //
       filterMap.put("/user/**","authc");
       bean.setFilterChainDefinitionMap(filterMap);
   
       // (2)设置拦截登录的请求(拦截之后跳转的页面)
       bean.setLoginUrl("/toLogin");
   
       return bean;
   }
   ```

   - `filterMap.put("/user/**","authc");`表示路径`user`下的所有请求必须通过认证才会跳转
   - `bean.setLoginUrl("/toLogin");`表示路径`user`下的所有请求会跳转到`URL`为`/toLogin`的登录页面进行认证

3. 测试

   点击`add`、`update`时，会跳转到登录界面(登录拦截)：

   ![image-20211227104718821](SpringBoot 基础.assets/image-20211227104718821.png)

   如果删除代码：

   ```java
   // (2)设置拦截登录的请求(拦截之后跳转的页面)
   bean.setLoginUrl("/toLogin");
   ```

   点击`add`、`update`时，跳转失败：

   ![image-20211227105005025](SpringBoot 基础.assets/image-20211227105005025.png)


## 用户认证

用户在跳转页面时需要经过验证，否则不能跳转到界面里，登录拦截实现了拦截功能，此时需要在登录界面输入相应的用户名和密码进行验证，来判断输入的用户是否可以通过验证并进行界面跳转。步骤如下：

1. 前端登录界面输入用户名和密码跳转到相应的`controller`进行处理。编写一个对应的`controller`

   ```java
   @RequestMapping("/login")
   public String login(String username, String password, Model model) {
   
       // 获取当前用户
       Subject subject = SecurityUtils.getSubject();
       // 封装用户的登录数据
       UsernamePasswordToken token = new UsernamePasswordToken(username, password);
       // 执行登录方法
       try {
           subject.login(token);
           return "index";
       } catch(UnknownAccountException e) { //用户名不存在
           model.addAttribute("msg","用户名错误");
           return "login";
       } catch (IncorrectCredentialsException e) { // 密码不存在
           model.addAttribute("msg","密码错误");
           return "login";
       }
   }
   ```

   此时进行登录测试会发现自动执行了`UserRealm`类中的`doGetAuthenticationInfo(...)`认证方法

   ![image-20211227153805951](SpringBoot 基础.assets/image-20211227153805951.png)

2. 在`UserRealm`类补全的`doGetAuthenticationInfo(...)`认证方法代码

   ```java
   // 认证
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
       System.out.println("执行了=>认证doGetAuthenticationInfo");
   
       // 模拟获取数据库中的用户名和密码
       String uname = "root";
       String pwd = "123456";
   
       UsernamePasswordToken userToken = (UsernamePasswordToken) token;
   
       System.out.println(userToken.getUsername());
   
       // 验证用户名是否输入正确
       if (!userToken.getUsername().equals(uname)) {
           // 用户名输入错误, return null 表示抛出异常 UnknownAccountException
           return null;
       }
   
       // 验证密码是否输入正确, 由 Shiro 自动完成
       return new SimpleAuthenticationInfo("", pwd, "");
   }
   ```

3. 用户名密码输入错误，需要将错误信息显示在登录界面，在登录代码login.html中写一个错误信息标签

   ```html
   <div class="field" style="align-content: center">
       <p th:text="${msg}" style="color: red"></p>
   </div>
   ```

4. 测试

   ![image-20211227163507053](SpringBoot 基础.assets/image-20211227163507053.png)



## 整合Mybatis

整合MyBatis流程如下：

1. `pom`文件导入`jdbc`、`Druid`、`Mybatis`、`log4j`依赖包，`yaml`文件中配置数据源参数

2. 编写`pojo`对象以及对应的`mapper`和`MapperXML`映射，`yaml`文件中整合`MyBatis`

3. 编写`service`层代码

4. 重写`UserRealm`类中的认证方法`doGetAuthenticationInfo(...)`

   ```java
   // 认证
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
       System.out.println("执行了=>认证doGetAuthenticationInfo");
       // 模拟获取数据库中的用户名和密码
       //String uname = "root";
       //String pwd = "123456";
   
       UsernamePasswordToken userToken = (UsernamePasswordToken) token;
   
       /*// 验证用户名是否输入正确
       if (!userToken.getUsername().equals(uname)) {
           // 用户名输入错误, return null 表示抛出异常 UnknownAccountException
           return null;
       }
       // 验证密码是否输入正确, 由 Shiro 自动完成
       return new SimpleAuthenticationInfo("", pwd, "");*/
   
       User user = userService.queryUserByName(userToken.getUsername());
   
       // 验证用户名是否输入正确
       if (user == null) {
           // 用户名输入错误, return null 表示抛出异常 UnknownAccountException
           return null;
       }
       // 验证密码是否输入正确, 由 Shiro 自动完成
       // user.getPwd() pwd 需要定义为 String 类型, 定义为 int 类型报错
       return new SimpleAuthenticationInfo("", user.getPwd(), "");
   }
   ```

   注意：`SimpleAuthenticationInfo()`中传入的密码需要`String`类型，传入`int`类型认证失败

## 用户授权

如果需要实现部分用户可以访问`add`页面，部分页面可以访问`update`页面，需要用到用户授权功能，实现步骤如下：

1. 在`ShiroConfig`配置类`getShiroFilterFactoryBean(...)`方法中添加如下代码：

   ```java
   // 授权
   // 携带 user:add 字符串的用户才能有权限访问 user 文件夹下的 add 页面
   filterMap.put("/user/add","perms[user:add]");
   filterMap.put("/user/update","perms[user:update]");
   ```

   表示如果想访问`/user`文件夹下的`add`资源，需要用户具有`user:add`权限

   之后登录认证成功之后输入正确的用户名和密码发现报错未授权，错误`401`

   ![image-20211228101430154](SpringBoot 基础.assets/image-20211228101430154.png)

   此时执行了`UserRealm`类中的授权方法

   ![image-20211228101621327](SpringBoot 基础.assets/image-20211228101621327.png)

   需要在`UserRealm`类中的授权方法编写授权代码

2. 用户未授权可以跳转到自定义未授权页面，而不是`401`页面

   `ShiroConfig`类`getShiroFilterFactoryBean(...)`方法添加：

   ```java
   // 设置未授权跳转页面
   bean.setUnauthorizedUrl("/unauth");
   ```

   类`MyController`添加：

   ```java
   @RequestMapping("/unauth")
   @ResponseBody
   public String unauth() {
       return "用户未授权访问权限!";
   }
   ```

   测试如果用户未授权跳转到以下界面：

   ![image-20211228112034387](SpringBoot 基础.assets/image-20211228112034387.png)

3. 在`UserRealm`类中的授权方法编写授权代码

   ```java
   // 授权
   @Override
   protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
       System.out.println("执行了=>授权doGetAuthorizationInfo");
   
       SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
       // 为所有用户增加 user:add 权限
       //info.addStringPermission("user:add");
   
       // 获取当前登录用户
       Subject subject = SecurityUtils.getSubject();
       // 获取认证方法中查到的当前登录用户信息 : User 对象
       User currentUser = (User) subject.getPrincipal();
       // 获取当前用户在数据库中查询到的拥有的权限, 并为当前用户设置该权限
       info.addStringPermission(currentUser.getPerms());
   
       return info;
   }
   ```
   
   - 对象`subject`通过`getPrincipal()`方法获得当前`User`用户，`User`用户的信息是在认证代码中数据库查询到的，修改认证代码如下所示，通过`return`返回了一个`SimpleAuthenticationInfo`对象，将`user`对象作为`SimpleAuthenticationInfo`对象的第一个参数传入，之后授权代码便可以通过`getPrincipal()`方法获得当前`User`用户的信息了。
   
     ```java
     // 认证
     @Override
     protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) {
     	// ...
         return new SimpleAuthenticationInfo(user, user.getPwd(), "");
     }
     ```
   
   - 通过`addStringPermission()`为当前`User`用户设置与数据库对应一致的权限，数据库表`usetest`增加权限字段`perms`，对应的User实体类也增加`perms`属性
   
     ```java
     @Data //Lombok标签
     @AllArgsConstructor
     @NoArgsConstructor
     public class User {
         //属性名称要与数据库对应表字段名称一致(不区分大小写)
         private int id;
         private String username;
         private String pwd;
         private String perms;
     }
     ```
   
     ![image-20211228155246787](SpringBoot 基础.assets/image-20211228155246787.png)



## 整合Thymeleaf

当某一个用户没有`update`权限时，希望将`update`隐藏，需要整合`Thymeleaf`，流程如下：



![image-20211228170240378](SpringBoot 基础.assets/image-20211228170240378.png)

1. 导入`Thymeleaf`与`Shiro`整合依赖包

   ```xml
   <!--thymeleaf整合shiro-->
   <dependency>
       <groupId>com.github.theborakompanioni</groupId>
       <artifactId>thymeleaf-extras-shiro</artifactId>
       <version>2.0.0</version>
   </dependency>
   ```

2. `Shiro`配置类`ShiroConfig`中进行配置

   ```java
   // 整合ShiroDialect : 用来整合 Shiro 和 Thymeleaf
   @Bean
   public ShiroDialect getShiroDialect() {
       return new ShiroDialect();
   }
   ```

3. 前端首页`index.html`代码导入命名空间

   ```xml
   <html lang="en" xmlns:th="http://www.thymeleaf.org"
         xmlns:shiro="http://www.thymeleaf.org/thymeleaf-extras-shiro">
   ```

4. 修改`index.html`代码

   ```html
   <body>
       <h1>首页</h1>
       <p th:text="${msg}"></p>
   
       <div>
           <a th:href="@{/toLogin}">登录</a>
       </div>
   
       <div shiro:hasPermission="user:add">
           <a th:href="@{/user/add}">add</a>
       </div>
       <div shiro:hasPermission="user:update">
           <a th:href="@{/user/update}">update</a>
       </div>
   </body>
   ```

   ![image-20211228172104304](SpringBoot 基础.assets/image-20211228172104304.png)

修改之后，测试发现，不具有`add`权限的用户登录，只显示`update`操作，此时已经登录成功，需要隐藏`登录`选项，通过获取用户`session`的操作来实现，步骤如下：

1. 认证方法中将已登录的用户信息保存在`session`中

   ```java
   // 认证
   @Override
   protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException { 
   	// ...
       
       // 获取当前 session, 并将当前用户信息存在 session 中
       Subject currentSubject = SecurityUtils.getSubject();
       Session session = currentSubject.getSession();
       session.setAttribute("loginUser", user);
       
       return new SimpleAuthenticationInfo(user, user.getPwd(), "");
   }
   ```

2. 修改首页`index.html`代码，拿到`session`信息，判断该用户是否存在，存在表示已经登录，隐藏`登录`选项

   ```html
   <div th:if="${session.loginUser==null}">
       <a th:href="@{/toLogin}">登录</a>
   </div>
   ```

最终测试发现登录成功之后，`登录`选项消失

![image-20211228173448668](SpringBoot 基础.assets/image-20211228173448668.png)



# Swagger

## Swagger简介

`Swagger`是一个规范和完整的框架，用于生成、描述、调用和可视化 `RESTful` 风格的 `Web` 服务；总体目标是使客户端和文件系统作为服务器以同样的速度来更新；文件的方法，参数和模型紧密集成到服务器端的代码，允许`API`来始终保持同步。主要用来：

- 接口文档在线自动生成
- 功能测试

`Swagger`是一组开源项目，其中主要要项目如下：

- `Swagger-tools`：提供各种与`Swagger`进行集成和交互的工具。例如模式检验、`Swagger 1.2`文档转换成`Swagger 2.0`文档等功能
- `Swagger-core`：用于`Java/Scala`的的`Swagger`实现。与`JAX-RS`(`Jersey`、`Resteasy`、`CXF`...)、`Servlets`和`Play`框架进行集成
- `Swagger-js`：用于`JavaScript`的`Swagger`实现
- `Swagger-node-express`：`Swagger`模块，用于`node.js`的`Express web`应用框架
- `Swagger-ui`：一个无依赖的`HTML`、`JS`和`CSS`集合，可以为`Swagger`兼容`API`动态生成优雅文档
- `Swagger-codegen`：一个模板驱动引擎，通过分析用户`Swagger`资源声明以各种语言生成客户端代码

## SpringBoot集成Swagger

创建新的`SpringBoot`项目`springboot-swagger-first`，集成`Swagger`，步骤如下：

1. `pom`文件添加`Swagger`依赖，需要使用到`Springfox-swagger2`、springfox-swagger-ui两个`jar`包

   ```xml
   <!--Swagger-->
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
   ```

   注意：

   - 要求`JDK1.8+`，否则`swagger2`无法运行
   - `Swagger`依赖要求`2.9.2`版本之前

2. 编写一个简单的`controller`

   ```java
   @Controller
   public class HelloController {
   
       @RequestMapping("/hello")
       @ResponseBody
       public String hello() {
           return "Hello, Swagger";
       }
   }
   ```

3. 编写`Swagger`配置类`SwaggerConfig`

   ```java
   @Configuration    // 配置类
   @EnableSwagger2   // 开启 swagger2 自动配置
   public class SwaggerConfig {
   }
   ```

4. 访问测试：http://localhost:8080/swagger-ui.html

   `Swagger 3.0.0`版本访问：http://localhost:8080/swagger-ui/index.html

   出现异常信息：`Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test (default-test) on project springboot-swagger-first: There are test failures.`

## 集成Swagger可能存在的问题

上述`SpringBoot`集成`Swagger`方式，可能会出现以下错误：

```text
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2021-12-22 21:53:13.203 ERROR 29012 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.context.ApplicationContextException: Failed to start bean 'documentationPluginsBootstrapper'; nested exception is java.lang.NullPointerException
	at org.springframework.context.support.DefaultLifecycleProcessor.doStart(DefaultLifecycleProcessor.java:181) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.context.support.DefaultLifecycleProcessor.access$200(DefaultLifecycleProcessor.java:54) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.context.support.DefaultLifecycleProcessor$LifecycleGroup.start(DefaultLifecycleProcessor.java:356) ~[spring-context-5.3.14.jar:5.3.14]
	at java.lang.Iterable.forEach(Iterable.java:75) ~[na:1.8.0_311]
	at org.springframework.context.support.DefaultLifecycleProcessor.startBeans(DefaultLifecycleProcessor.java:155) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.context.support.DefaultLifecycleProcessor.onRefresh(DefaultLifecycleProcessor.java:123) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:935) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:586) ~[spring-context-5.3.14.jar:5.3.14]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:145) ~[spring-boot-2.6.2.jar:2.6.2]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:730) [spring-boot-2.6.2.jar:2.6.2]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:412) [spring-boot-2.6.2.jar:2.6.2]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:302) [spring-boot-2.6.2.jar:2.6.2]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1301) [spring-boot-2.6.2.jar:2.6.2]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1290) [spring-boot-2.6.2.jar:2.6.2]
	at com.lin.swagger.SwaggerDemoApplication.main(SwaggerDemoApplication.java:13) [classes/:na]
Caused by: java.lang.NullPointerException: null
	at springfox.documentation.spi.service.contexts.Orderings$8.compare(Orderings.java:112) ~[springfox-spi-2.9.2.jar:null]
	at springfox.documentation.spi.service.contexts.Orderings$8.compare(Orderings.java:109) ~[springfox-spi-2.9.2.jar:null]
	at com.google.common.collect.ComparatorOrdering.compare(ComparatorOrdering.java:37) ~[guava-20.0.jar:na]
	at java.util.TimSort.countRunAndMakeAscending(TimSort.java:355) ~[na:1.8.0_311]
	at java.util.TimSort.sort(TimSort.java:220) ~[na:1.8.0_311]
	at java.util.Arrays.sort(Arrays.java:1438) ~[na:1.8.0_311]
	at com.google.common.collect.Ordering.sortedCopy(Ordering.java:855) ~[guava-20.0.jar:na]
	at springfox.documentation.spring.web.plugins.WebMvcRequestHandlerProvider.requestHandlers(WebMvcRequestHandlerProvider.java:57) ~[springfox-spring-web-2.9.2.jar:null]
	at springfox.documentation.spring.web.plugins.DocumentationPluginsBootstrapper$2.apply(DocumentationPluginsBootstrapper.java:138) ~[springfox-spring-web-2.9.2.jar:null]
	at springfox.documentation.spring.web.plugins.DocumentationPluginsBootstrapper$2.apply(DocumentationPluginsBootstrapper.java:135) ~[springfox-spring-web-2.9.2.jar:null]
	at com.google.common.collect.Iterators$7.transform(Iterators.java:750) ~[guava-20.0.jar:na]
	at com.google.common.collect.TransformedIterator.next(TransformedIterator.java:47) ~[guava-20.0.jar:na]
	at com.google.common.collect.TransformedIterator.next(TransformedIterator.java:47) ~[guava-20.0.jar:na]
	at com.google.common.collect.MultitransformedIterator.hasNext(MultitransformedIterator.java:52) ~[guava-20.0.jar:na]
	at com.google.common.collect.MultitransformedIterator.hasNext(MultitransformedIterator.java:50) ~[guava-20.0.jar:na]
	at com.google.common.collect.ImmutableList.copyOf(ImmutableList.java:249) ~[guava-20.0.jar:na]
	at com.google.common.collect.ImmutableList.copyOf(ImmutableList.java:209) ~[guava-20.0.jar:na]
	at com.google.common.collect.FluentIterable.toList(FluentIterable.java:614) ~[guava-20.0.jar:na]
	at springfox.documentation.spring.web.plugins.DocumentationPluginsBootstrapper.defaultContextBuilder(DocumentationPluginsBootstrapper.java:111) ~[springfox-spring-web-2.9.2.jar:null]
	at springfox.documentation.spring.web.plugins.DocumentationPluginsBootstrapper.buildContext(DocumentationPluginsBootstrapper.java:96) ~[springfox-spring-web-2.9.2.jar:null]
	at springfox.documentation.spring.web.plugins.DocumentationPluginsBootstrapper.start(DocumentationPluginsBootstrapper.java:167) ~[springfox-spring-web-2.9.2.jar:null]
	at org.springframework.context.support.DefaultLifecycleProcessor.doStart(DefaultLifecycleProcessor.java:178) ~[spring-context-5.3.14.jar:5.3.14]
	... 14 common frames omitted
```

提取关键错误信息：

```text
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
...
Caused by: java.lang.NullPointerException: null
```

原因是在`springboot2.6.0`中将`SpringMVC` 默认路径匹配策略从`AntPathMatcher` 更改为`PathPatternParser`，导致出错。可以在启动类上加上`@EnableWebMvc`注解或者在配置中切换为原先的`AntPathMatcher`：`spring.mvc.pathmatch.matching-strategy=ant_path_matcher`

```java
@SpringBootApplication
@EnableWebMvc
public class SpringbootSwaggerFirstApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootSwaggerFirstApplication.class, args);
    }
}
```

加上以上配置后启动就不报错了，但是访问http://localhost:8080/swagger-ui/index.html报`404`

解决办法：把原来的`swagger2`和`swagger-ui`依赖删掉，改成`spring-boot-starter`依赖：

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-boot-starter</artifactId>
	<version>3.0.0</version>
</dependency>
```

![image-20211229154414504](SpringBoot 基础.assets/image-20211229154414504.png)

## 配置Swagger

`Swagger`的实例`Bean`是一个`Docket`对象，所以通过配置`Docket`实例来配置`Swagger`，步骤如下：

1. `SwaggerConfig`配置类生成`Docket`实例

   ```java
   @Bean
   public Docket docket() {
       return new Docket(DocumentationType.SWAGGER_2);
   }
   ```

2. 通过`apiInfo()`方法配置文档信息

   ```java
   // 配置文档信息
   public ApiInfo apiInfo() {
       // contact 联系人信息
       Contact contact = new Contact("联系人名字", "http://xxx.xxx.com/联系人访问链接", "联系人邮箱"); 
       return new ApiInfo(
           "Swagger学习", // 标题
           "学习演示如何配置Swagger", // 描述
           "v1.0", // 版本
           "http://terms.service.url/组织链接", // 组织链接
           contact, // 联系人信息
           "Apach 2.0 许可", // 许可
           "许可链接", // 许可连接
           new ArrayList<>() // 扩展
       );
   }
   ```

3. 修改`docket()`方法，`Docket` 实例关联`apiInfo()`

   ```java
   @Bean
   public Docket docket() {
       return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo());
   }
   ```

最终测试结果如下：

![image-20211229160906753](SpringBoot 基础.assets/image-20211229160906753.png)

部分源码解析：

```java
public Docket(DocumentationType documentationType) {
  this.documentationType = documentationType;
}
```

```java
public class DocumentationType extends SimplePluginMetadata {
  public static final DocumentationType SWAGGER_12 = new DocumentationType("swagger", "1.2");
  public static final DocumentationType SWAGGER_2 = new DocumentationType("swagger", "2.0");
  public static final DocumentationType OAS_30 = new DocumentationType("openApi", "3.0");
  // ...
}
```

```java
public ApiInfo(String title, String description, String version, String termsOfServiceUrl, Contact contact, String license, String licenseUrl, Collection<VendorExtension> vendorExtensions) {
    this.title = title;
    this.description = description;
    this.version = version;
    this.termsOfServiceUrl = termsOfServiceUrl;
    this.contact = contact;
    this.license = license;
    this.licenseUrl = licenseUrl;
    this.vendorExtensions = new ArrayList(vendorExtensions);
}
```

```java
public Contact(String name, String url, String email) {
    this.name = name;
    this.url = url;
    this.email = email;
}
```

## 配置扫描接口

`Swagger`可以指定配置需要扫描的接口，步骤如下：

1. 使用`select()`方法配置需要扫描的接口

   ```java
   @Bean
   public Docket docket() {
       return new Docket(DocumentationType.SWAGGER_2).
           apiInfo(apiInfo()).
           select().
   		// 通过 .select() 方法去配置扫描接口, RequestHandlerSelectors 配置如何扫描接口
           apis(RequestHandlerSelectors.basePackage("com.swagger.first.controller")).
           build();
   }
   ```

2. 测试，发现`Swagger`仅仅扫描了包路径`com.swagger.first.controller`下的内容

   ![image-20211229165140626](SpringBoot 基础.assets/image-20211229165140626.png)

3. 除了通过包路径配置扫描接口外，还可以通过配置其他方式扫描接口，配置方式如下：

   ```java
   @Bean
   public Docket docket() {
       return new Docket(DocumentationType.SWAGGER_2).
           apiInfo(apiInfo()).
           select().
           // 通过 .select() 方法去配置扫描接口, RequestHandlerSelectors 配置如何扫描接口
           apis(RequestHandlerSelectors.basePackage("com.swagger.first.controller")).
           // apis(RequestHandlerSelectors.any()). // 扫描所有, 项目中的所有接口都会被扫描到
           // apis(RequestHandlerSelectors.none()). // 不扫描接口
           // 通过类上的注解扫描, Controller.class : 只扫描有 @Controller 注解的类
           // apis(RequestHandlerSelectors.withClassAnnotation(Controller.class)).
           // 通过方法上的注解扫描, GetMapping.class : 只扫描 get 请求, 方法上有注解 @GetMapping()
           // apis(RequestHandlerSelectors.withMethodAnnotation(GetMapping.class)).
           build();
   }
   ```

4. 可以配置接口扫描过滤：

   ```java
   @Bean
   public Docket docket() {
       return new Docket(DocumentationType.SWAGGER_2).
           apiInfo(apiInfo()).
           select().
           // 配置通过 path 过滤, 即这里只扫描请求以 /hello 开头的接口
           paths(PathSelectors.ant("/hello/**")).
           build();
   }
   
   // path 过滤 : paths(PathSelectors.ant("/hello/**")), HelloController 可以被扫描到
   @Controller
   public class HelloController {
   
       @RequestMapping("/hello")
       @ResponseBody
       public String hello() {
           return "Hello, Swagger";
       }
   }
   ```

   如果修改为：

   ```java
   paths(PathSelectors.ant("/wyj/**"))
   ```

   由于代码中没有`/wyj`请求，因此扫描不到任何接口信息：

   ![image-20211229171818976](SpringBoot 基础.assets/image-20211229171818976.png)

   `PathSelectors`常用方法有：

   ```java
   any() // 任何请求都扫描
   none() // 任何请求都不扫描
   regex(final String pathRegex) // 通过正则表达式控制
   ant(final String antPattern) // 通过ant()控制
   ```

## 配置Swagger开关

当项目在开发环境`dev`和测试环境`test`时，需要开启`swagger`，项目发布环境`pro`不需要显示`swagger`，可以通过配置`swagger`开关来实现，步骤如下：

1. 通过`enable()`方法配置是否启用`swagger`，如果是`false`，`swagger`将不能在浏览器中访问

   ```java
   @Bean
   public Docket docket() {
       return new Docket(DocumentationType.SWAGGER_2).
           apiInfo(apiInfo()).
           // 配置是否启用 swagger, 如果是 false, 浏览器将无法访问
           enable(false).
           select().
           apis(RequestHandlerSelectors.basePackage("com.swagger.first.controller")).
           build();
   }
   ```

2. 测试

   ![image-20211229172351726](SpringBoot 基础.assets/image-20211229172351726.png)

3. 调用`Docket`方法`docket(Environment environment)` 方法

   ```java
   @Bean
   public Docket docket(Environment environment) {
   
       // 设置项目处于哪些环境时, 需要开启 swagger
       // Profiles.of("dev", "test") 处于 dev test 环境 开启 swagger
       Profiles of = Profiles.of("dev", "test");
       // 判断当前项目所处环境, 如果为 dev test 环境, 返回 true
       // 通过 enable() 接收此参数判断并决定是否显示
       boolean b = environment.acceptsProfiles(of);
   
       return new Docket(DocumentationType.SWAGGER_2).
               apiInfo(apiInfo()).
               enable(b).
               select().
               apis(RequestHandlerSelectors.basePackage("com.swagger.first.controller")).
               build();
   }
   ```
   
4. 为每一个测试环境写一个`yaml`文件

   `dev`环境`application-dev.yaml`：

   ```yaml
   server:
     port: 8081
   ```

   `test`环境`application-test.yaml`：

   ```yaml
   server:
     port: 8082
   ```

   `pro`环境`application-pro.yaml`：

   ```yaml
   server:
     port: 8080
   ```

5. 测试

   在`application.yaml`中激活`dev`环境：

   ```yaml
   # 激活 dev 环境
   spring:
     profiles:
       active: dev
   ```

   访问http://localhost:8081/swagger-ui/index.html，发现开启了`swagger`，结果如下：

   ![image-20211230093534012](SpringBoot 基础.assets/image-20211230093534012.png)

   使用`pro`环境测试，发现`swagger`关闭：

   ![image-20211230093653564](SpringBoot 基础.assets/image-20211230093653564.png)

## 配置API分组

如果多人协同开发一个项目，不同的开发者负责不同的接口，可以使用`Docket`中`groupName(groupName)`进行分组，创建多个`Docket`实例（每个实例代表一个分组）即可，流程如下：

1. Swagger默认分组：

   ![image-20211230094031359](SpringBoot 基础.assets/image-20211230094031359.png)

2. 使用`groupName`修改分组名字：

   ```java
   @Bean
   public Docket docket(Environment environment) {
   
   	// ...
       return new Docket(DocumentationType.SWAGGER_2).
               apiInfo(apiInfo()).
               groupName("王宇").
               // ...
   }
   ```

   测试：

   ![image-20211230094247348](SpringBoot 基础.assets/image-20211230094247348.png)

3. 创建多个分组，为每一个分组创建一个`Docket`实例即可：

   ```java
   @Bean
   public Docket docket1(Environment environment) {
       return new Docket(DocumentationType.SWAGGER_2).
               apiInfo(apiInfo()).
               groupName("张宇");
   }
   
   @Bean
   public Docket docket2(Environment environment) {
       return new Docket(DocumentationType.SWAGGER_2).
               apiInfo(apiInfo()).
               groupName("刘宇");
   }
   ```

   测试结果：

   ![image-20211230095335202](SpringBoot 基础.assets/image-20211230095335202.png)

## 常用注解

`Swagger`的所有注解定义在`io.swagger.annotations`包下，常用注解如下：

| `Swagger`注解                                            | 简单说明                                                 |
| :------------------------------------------------------- | -------------------------------------------------------- |
| `@Api(tags = "xxx模块说明")`                             | 作用在模块类上                                           |
| `@ApiOperation("xxx接口说明")`                           | 作用在接口方法上                                         |
| `@ApiModel("xxxPOJO说明")`                               | 作用在模型类上：如`VO`、`BO`                             |
| `@ApiModelProperty(value = "xxx属性说明",hidden = true)` | 作用在类方法和属性上，`hidden`设置为`true`可以隐藏该属性 |
| `@ApiParam("xxx参数说明")`                               | 作用在参数、方法和字段上，类似`@ApiModelProperty`        |

测试如下：

1. 编写`User`实体类和接口`HelloController`相关方法（这里的接口表示一个`controller`类）

   ```java
   @Controller
   public class HelloController {
   
       @GetMapping("/hello")
       @ResponseBody
       public String hello() {
           return "Hello, Swagger";
       }
   
       // 只要接口中返回值中存在实体类, 就会被扫描到 Swagger 中
       @PostMapping("/user")
       public User user() {
           return new User();
       }
   
       @ApiOperation("Hello2控制接口")
       @GetMapping("/hello2")
       public String hello2(@ApiParam("用户名") String username) {
           return "hello" + username;
       }
   
   }
   ```

   ```java
   @Api
   @ApiModel("用户实体类-User")
   public class User {
       @ApiModelProperty("用户名")
       private String username;
       @ApiModelProperty("密码")
       private String password;
   
       @ApiModelProperty("获取用户名")
       public String getUsername() {
           return username;
       }
   
       public void setUsername( @ApiParam("方法参数-用户名") String username) {
           this.username = username;
       }
   
       public String getPassword() {
           return password;
       }
   
       public void setPassword(String password) {
           this.password = password;
       }
   }
   ```

2. 测试

   ![image-20211230111015060](SpringBoot 基础.assets/image-20211230111015060.png)
   
   在实体类`User`中，需要注意以下两点：
   
   - 并不是因为`@ApiModel`这个注解让实体显示在这里了，而是只要出现在接口方法的返回值上的实体都会显示在这里，而`@ApiModel`和`@ApiModelProperty`这两个注解只是为实体添加注释的
   
   - `@ApiModelProperty`、`@ApiParam`不起作用，在`controller`接口中起作用，如下：
   
   ![image-20211230111041570](SpringBoot 基础.assets/image-20211230111041570.png)
   
   ![image-20211230111543667](SpringBoot 基础.assets/image-20211230111543667.png)

# 任务

## 异步任务

需求：在网站上发送邮件，后台会去发送邮件，此时前台会造成响应不动，造成响应不动，直到邮件发送完毕，响应才会成功，所以一般会采用多线程的方式去处理类似任务。创建新的项目`springboot-task-first`，实现异步任务，流程如下：

1. 编写`Service`类`AsyncService`，该类实现一个`hello()`方法，模拟进行处理数据，需要用时3秒

   ```java
   @Service
   public class AsyncService {
       public void hello(){
           try {
               Thread.sleep(3000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println("数据处理中....");
       }
   }
   ```

2. 编写`controller`类`AsyncController`，调用`Service`层`hello()`方法

   ```java
   @RestController
   public class AsyncController {
   
       @Autowired
       AsyncService asyncService;
   
       @RequestMapping("/hello")
       public String hello(){
           asyncService.hello(); // 调用方法, 等待 3 秒
           return "OK";
       }
   }
   ```

3. 测试

   发现前端需要等待3秒才会相应，可以使用多线程的方式让用户先得到消息，之后后台线程对数据进行处理，通过`@Async`注解实现，代码如下：

4. 在需要处理数据的代码上加上`@Async`注解，在主程序上添加一个注解`@EnableAsync` ，开启异步注解功能

   ```java
   // 异步方法
   @Async
   public void hello(){
       try {
           Thread.sleep(3000);
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
       System.out.println("数据处理中....");
   }
   ```

   ```java
   @SpringBootApplication
   @EnableAsync
   public class SpringbootTaskFirstApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(SpringbootTaskFirstApplication.class, args);
       }
   
   }
   ```

   重启测试，网页瞬间响应，后台代码在异步执行`hello()`方法

## 邮件任务

`SpringBoot`发送邮件操作流程如下：

1. `pom`文件引入依赖：

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-mail</artifactId>
   </dependency>
   ```

2. `application.properties`配置文件：

   ```properties
   spring.mail.username=1013801973@qq.com
   spring.mail.password=bhhfkgvrzfuibaja
   spring.mail.host=smtp.qq.com
   # QQ 邮箱需要配置 ssl
   spring.mail.properties.mail.smtp.ssl.enable=true
   ```

   `password`为自己邮箱开启`SMTP`服务的授权码，需要要个人邮箱开启服务

   ![image-20211231094853736](SpringBoot 基础.assets/image-20211231094853736.png)

3. 测试

   ```java
   @SpringBootTest
   class SpringbootTaskFirstApplicationTests {
   
       @Autowired
       JavaMailSenderImpl javaMailSender;
   
       // 发送简单邮件
       @Test
       void contextLoads() {
           SimpleMailMessage mailMessage = new SimpleMailMessage();
   
           mailMessage.setSubject("狂神，你好");
           mailMessage.setText("谢谢你的狂神说Java系列课程");
   
           mailMessage.setTo("1637317579@qq.com");
           mailMessage.setFrom("1013801973@qq.com");
           javaMailSender.send(mailMessage);
       }
   
       // 发送复杂邮件
       @Test
       void contextLoads2() throws MessagingException {
           // 创建复杂邮件
           MimeMessage mimeMessage = javaMailSender.createMimeMessage();
           // 组装 true-表示是否支持多消息发送, 文本、附件、内联元素等...
           MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
   
           //正文 true-表示是否支持 html 内容文本发送
           helper.setSubject("狂神，你好~plus");
           helper.setText("<p style='color:red'>谢谢你的狂神说Java系列课程</p>", true);
   
           //附件
           helper.addAttachment("1.jpg", new File("D:\\Program Files\\workspace\\idea\\1.jpg"));
           helper.addAttachment("2.jpg", new File("D:\\Program Files\\workspace\\idea\\2.jpg"));
   
           helper.setTo("1637317579@qq.com");
           helper.setFrom("1013801973@qq.com");
   
           javaMailSender.send(mimeMessage);
   
       }
   }
   ```

   ![image-20211231103203307](SpringBoot 基础.assets/image-20211231103203307.png)

   ![image-20211231103321612](SpringBoot 基础.assets/image-20211231103321612.png)

## 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨分析一次前一天的日志信息，`Spring`提供了异步执行任务调度的方式来实现定时任务。需要使用到两个接口：

- `TaskExecutor`接口：任务执行者
- `TaskScheduler`接口：任务调度者

两个注解：

- `@EnableScheduling`：开启定时功能
- `@Scheduled`：定义执行时间

`cron`表达式（定义定时任务执行规则，表达式生成器网址 http://www.bejson.com/othertools/cron/）：

| 字段 |        允许值         |  允许特殊字符   |
| :--: | :-------------------: | :-------------: |
|  秒  |         0-59          |     , - * /     |
|  分  |         0-59          |     , - * /     |
| 小时 |         0-23          |     , - * /     |
| 日期 |         1-31          | , - * / ? L W C |
| 月份 |         1-12          |     , - * /     |
| 星期 | 0-1或SUN-SAT 0,7是SUN | , - * / ? L W C |

| 特殊字符 |          代表含义          |
| :------: | :------------------------: |
|    ,     |            枚举            |
|    -     |            区间            |
|    *     |            任意            |
|    /     |            步长            |
|    ?     |      日/星期冲突匹配       |
|    L     |            最后            |
|    W     |           工作日           |
|    C     | 和calendar练习后计算过的值 |
|    #     |   星期，4#2 第二个星期三   |

测试步骤如下：

1. 创建一个`ScheduledService`类，定义一个需要定时执行的`hello()`方法

   ```java
   @Service
   public class ScheduledService {
       // 在一个特定的时间执行这个方法——Timer
       // cron表达式
       // 秒 分 时 日 月 周几
   
       /*
           0 49 11 * * ?       每天的 11 点 49 分 00 秒执行
           0 0/5 11,12 * * ?   每天的 11 点和 12 点每个五分钟执行一次
           0 15 10 ? * 1-6     每个月的周一到周六的 10 点 15 分执行一次
           0/2 * * * * ?       每 2 秒执行一次
        */
       @Scheduled(cron = "0/2 * * * * ?")
       public void hello() {
           System.out.println("hello,你被执行了");
       }
   }
   ```

2. 在主程序上增加`@EnableScheduling` 开启定时任务功能

   ```java
   @SpringBootApplication
   @EnableAsync //开启异步注解功能
   @EnableScheduling //开启基于注解的定时任务
   public class SpringbootTaskFirstApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(SpringbootTaskFirstApplication.class, args);
       }
   }
   ```

3. 测试，`hello()`方法每隔 2 秒被执行一次：

   ![image-20211231104900760](SpringBoot 基础.assets/image-20211231104900760.png)

4. 常用的表达式：

   ```text
   （1）0/2 * * * * ?   表示每2秒 执行任务
   （1）0 0/2 * * * ?   表示每2分钟 执行任务
   （1）0 0 2 1 * ?   表示在每月的1日的凌晨2点调整任务
   （2）0 15 10 ? * MON-FRI   表示周一到周五每天上午10:15执行作业
   （3）0 15 10 ? 6L 2002-2006   表示2002-2006年的每个月的最后一个星期五上午10:15执行作
   （4）0 0 10,14,16 * * ?   每天上午10点，下午2点，4点
   （5）0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时
   （6）0 0 12 ? * WED   表示每个星期三中午12点
   （7）0 0 12 * * ?   每天中午12点触发
   （8）0 15 10 ? * *   每天上午10:15触发
   （9）0 15 10 * * ?     每天上午10:15触发
   （10）0 15 10 * * ?   每天上午10:15触发
   （11）0 15 10 * * ? 2005   2005年的每天上午10:15触发
   （12）0 * 14 * * ?     在每天下午2点到下午2:59期间的每1分钟触发
   （13）0 0/5 14 * * ?   在每天下午2点到下午2:55期间的每5分钟触发
   （14）0 0/5 14,18 * * ?     在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
   （15）0 0-5 14 * * ?   在每天下午2点到下午2:05期间的每1分钟触发
   （16）0 10,44 14 ? 3 WED   每年三月的星期三的下午2:10和2:44触发
   （17）0 15 10 ? * MON-FRI   周一至周五的上午10:15触发
   （18）0 15 10 15 * ?   每月15日上午10:15触发
   （19）0 15 10 L * ?   每月最后一日的上午10:15触发
   （20）0 15 10 ? * 6L   每月的最后一个星期五上午10:15触发
   （21）0 15 10 ? * 6L 2002-2005   2002年至2005年的每月的最后一个星期五上午10:15触发
   （22）0 15 10 ? * 6#3   每月的第三个星期五上午10:15触发
   ```

# 分布式入门

## 分布式概述

分布式系统是由一组通过网络进行通信、为了完成共同的任务而协调工作的计算机节点组成的系统。分布式系统的出现是为了用廉价的、普通的机器完成单个计算机无法完成的计算、存储任务。其目的是**利用更多的机器，处理更多的数据**。

首先需要明确的是，只有当单个节点的处理能力无法满足日益增长的计算、存储任务的时候，且硬件的提升（加内存、加磁盘、使用更好的`CPU`）高昂到得不偿失的时候，应用程序也不能进一步优化的时候，我们才需要考虑分布式系统。因为，分布式系统要解决的问题本身就是和单机系统一样的，而由于分布式系统多节点、通过网络通信的拓扑结构，会引入很多单机系统没有的问题，为了解决这些问题又会引入更多的机制、协议，带来更多的问题。

**分布式与集群的区别**：

- 分布式：不同的业务模块部署在不同的服务器上或者同一个业务模块分拆多个子业务，部署在不同的服务器上，解决高并发的问题
- 集群：同一个业务部署在多台机器上，提高系统可用性

## CAP理论

**概述：**

`CAP`理论是说对于分布式数据存储，最多只能同时满足一致性（`C`，`Consistency`）、可用性（`A`， `Availability`）、分区容错性（P，`Partition Tolerance`）中的两者。

- 一致性：是指对于每一次读操作，要么都能够读到最新写入的数据，要么错误
- 可用性：是指对于每一次请求，都能够得到一个及时的、非错的响应，但是不保证请求的结果是基于最新写入的数据
- 分区容错性：是指由于节点之间的网络问题，即使一些消息对包或者延迟，整个系统能继续提供服务（提供一致性或者可用性）

**适用场景：**

>  在分布式系统中，围绕着`CAP`理论，主要关注点就是复制，一致性，容错性

1. 复制：保证数据高可用性

   为了保证系统的高可用和高可靠性，通过复制的方式，让数据在系统中存储多个副本。以服务实例多副本为例，当一个服务发生异常时，客户端就直接调用其他正常的副本。

   ![image-20220106085408294](SpringBoot 基础.assets/image-20220106085408294.png)

2. 一致性

   在数据的复制中，由于存在多个数据副本，就会存在主数据与副本数据一致性的问题。在同一份数据的副本中，一般有一个副本为主副本，其他的备副本。在数据的复制过程中，复制的方式分为两种分别如下：

   - 强同步复制，数据的写操作需要同步到主副本和所有的备副本，并且全部写入成功后，才返回成功状态。这样，当系统出现异常时，切换到其他任何一个备份副本时，数据是一致的。但是，强同步复制性能不好，而且可用性比较差。如果，在复制过程中，如果某个备份节点出现故障，这时，会阻塞数据的正常写服务。
   - 异步复制，当数据写入操作成功后，当数据成功复制到主副本时，甚至还没复制时，写操作就返回成功状态。这样，异步复制的性别比较好，但是，当主备出现故障时可能出现数据丢失

3. 容错性

   分布式系统中，集群的规模越大发生错误的概率就也大。一般，分布式系统发生异常时，都能够自动容错，保证系统的高可用

## RPC

`RPC`（`Remote Procedure  Call`）是指远程过程调用，是一种进程间通信方式，他是一种技术的思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。

也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。为什么要用RPC呢？就是无法在一个进程内，甚至一个计算机内通过本地调用的方式完成的需求，比如不同的系统间的通讯，甚至不同的组织间的通讯，由于计算能力需要横向扩展，需要在多台机器组成的集群上部署应用。RPC就是要像调用本地的函数一样去调远程函数。

**RPC基本原理：**

核心模块：通讯，序列化

![image-20220106094701589](SpringBoot 基础.assets/image-20220106094701589.png)

![image-20220106102044853](SpringBoot 基础.assets/image-20220106102044853.png)

参考文章：

<https://www.jianshu.com/p/2accc2840a1b>

<https://www.jianshu.com/p/b0343bfd216e>

## Dubbo概述

### Dubbo产生背景

`Dubbo`官方文档：<https://dubbo.apache.org/zh/>

互联网的发展过程中，随着流量的增大，常规的垂直应用架构已无法应对应用场景，架构就发生了演变：

- 单一的应用架构：网站流量很小，只需一个应用，将所有功能都部署在一起
- 应用和数据库单独部署
-  应用和数据库集群部署
- 数据库压力变大，读写分离
- 使用缓存技术加快速度
- 数据库分库分表
- 应用分为不同的类型拆分

当应用架构发展到此阶段时，应用与应用之间的关系会变得十分复杂，随之出现以下几个问题：

- 当服务越来越多时，服务 `URL` 配置管理变得非常困难，`F5` 硬件负载均衡器的单点压力也越来越大
-  当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系
- 接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？

为了解决上述由于架构的演变所产生的问题几个问题，于是，`Dubbo` 随之产生。

### Dubbo技术架构

`Dubbo`服务架构如下图所示：

![image-20220106091629679](SpringBoot 基础.assets/image-20220106091629679.png)

**节点角色说明:**

| 节点      | 角色说明                                                     |
| :-------- | :----------------------------------------------------------- |
| Provider  | **暴露服务的服务提供方**，服务提供者在启动时，向注册中心注册自己提供的服务 |
| Consumer  | **调用远程服务的服务消费方**，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用 |
| Registry  | **服务注册与发现的注册中心**，注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者 |
| Monitor   | 统计服务的调用次数和调用时间的监控中心                       |
| Container | 服务运行容器                                                 |

类似**生产者-消费者**模型，在这种模型上，加上了**注册中心和监控中心**，用于管理提供方提供的`URL`，以及管理整个过程，**流程如下**：

- 启动容器，加载，运行服务提供者
- 服务提供者在启动时，在注册中心**发布注册**自己提供的服务
- 服务消费者在启动时，在注册中心**订阅**自己所需的服务
- 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
- 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用
- 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

**`Zookeeper`是`Dubbo`推荐的注册中心**

## Zookeeper下载与安装

`Zookeeper`是`Dubbo`推荐的注册中心，`Windows`下安装流程如下：

1. 官网：<https://zookeeper.apache.org/releases.html>下载`bin.tar.gz`压缩包解压，经过编译的可以直接解压使用

2. 进入解压目录conf文件夹，复制`zoo_sample.cfg`副本并重新命名为`zoo.cfg`

   ![image-20220106145208296](SpringBoot 基础.assets/image-20220106145208296.png)

3.  以管理员身份运行`/bin/zkServer.cmd` 

   > 初次运行可能报错，在/bin文件夹下打开`zkServer.cmd`末尾添加`pause`启动，观察报错原因并处理

   ![image-20220106145535342](SpringBoot 基础.assets/image-20220106145535342.png)

   > 添加`pause`指令仍然闪退，可能由于`JDK`版本过低，建议下载低版本`Zookeeper`，实测JDK1.8 `Zookeeper 3.5.9`可用

4. 修改`zoo.cfg`配置文件

   `zookeeper 3.5`之后有个内嵌的管理控制台是通过`Jetty`启动，会占用 8080 端口，需要修改配置文件：

   ```apl
   admin.serverPort=8888
   ```

   ![image-20220105113449621](SpringBoot 基础.assets/image-20220105113449621.png)`zoo.cfg`配置文件分析：

   ```apl
   tickTime=2000  # Zookeeper 中最小的时间单位长度(ms)
   
   initLimit=10  # follower 节点启动后与 leader 节点完成数据同步的时间
   
   syncLimit=5 # leader 节点和 follower 节点进行心跳检测的最大延时时间
   
   dataDir=/tmp/zookeeper  # 表示 Zookeeper 服务器存储快照文件的目录
   
   dataLogDir # 表示配置 Zookeeper事务日志的存储路径, 默认指定在 dataDir 目录下 
   
   clientPort # 表示客户端和服务端建立连接的端口号： 2181
   ```

   ![image-20220105102005854](SpringBoot 基础.assets/image-20220105102005854.png)

5. 启动cli.cmd测试

   ![image-20220105101911556](SpringBoot 基础.assets/image-20220105101911556.png)

##  安装dubbo-admin

`dubbo-admin`是`Dubbo RPC`框架的管理端，可以对注册的服务(`provider`)和服务调用方(`comsumer`)进行服务治理，包括路由、监控、配置等功能，**可以对注册到`zookeeper`注册中心的服务或服务消费者进行管理**。安装流程如下：

1. 下载解压，`dubbo-admin`安装地址：<https://github.com/apache/dubbo-admin/tree/master>

   > 旧版本下载`git`命令： `git clone -b master.0.2.0 https://github.com/apache/dubbo-admin.git`

2. 修改 `dubbo-admin\src\main\resources \application.properties` 指定`Zookeeper`地址

   ```properties
   server.port=7001
   spring.velocity.cache=false
   spring.velocity.charset=UTF-8
   spring.velocity.layout-url=/templates/default.vm
   spring.messages.fallback-to-system-locale=false
   spring.messages.basename=i18n/message
   spring.root.password=root
   spring.guest.password=guest
   
   dubbo.registry.address=zookeeper://127.0.0.1:2181
   ```

   ![image-20220106152516905](SpringBoot 基础.assets/image-20220106152516905.png)

   > 旧版本`dubbo-admin`启动端口号为7001，新版本为8080，新版本同时需要修改启动端口号

3. 在项目目录下窗口打包`dubbo-admin`

   ```
   mvn clean package -Dmaven.test.skip=true
   ```

   ![image-20220106153130974](SpringBoot 基础.assets/image-20220106153130974.png)

4. 打开`Zookeeper`服务，执行`dubbo-admin\target` 下的`dubbo-admin-0.0.1-SNAPSHOT.jar`

   ```
   java -jar dubbo-admin-0.0.1-SNAPSHOT.jar
   ```

5. 测试，访问<http://localhost:7001/>，输入登录账户和密码，默认都为root

   ![image-20220106153509959](SpringBoot 基础.assets/image-20220106153509959.png)

## SpringBoot整合Dubbo+Zookeeper

`SpringBoot`整合`Dubbo`+`Zookeeper`流程如下：

1. `idea`创建一个空项目`dubbo-zookeeper`，空项目中分别创建两个`Mudules` -`Springboot`应用：`provider-server`、`consumer-server`：

   ![image-20220106153838158](SpringBoot 基础.assets/image-20220106153838158.png)

   ![image-20220106154117713](SpringBoot 基础.assets/image-20220106154117713.png)

2. 编写服务提供者代码并测试，移步[服务提供者](#_link_2nd)  

3. 编写服务消费者代码并测试，移步[服务消费者](#_link_3rd)

### <a id=_link_2nd>服务提供者</a>

**将服务提供者注册到注册中心，需要整合Dubbo和Zookeeper包**，服务提供者代码实现如下（举例卖票服务）：

1. 编写接口及其实现类

   ```java
   public interface TicketService {
       public String getTicket();
   }
   
   @Component  
   public class TicketServiceImpl implements TicketService {
       @Override
       public String getTicket() {
           return "服务商提供卖票服务";
       }
   }
   ```

2. 导入相关依赖包：

   ```xml
   <!-- Dubbo整合SpringBoot依赖包 dubbo-springboot-->
   <dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>2.7.3</version>
   </dependency>
   
   <!-- Zookeeper服务端依赖包 zkclient-->
   <dependency>
       <groupId>com.github.sgroschupf</groupId>
       <artifactId>zkclient</artifactId>
       <version>0.1</version>
   </dependency>
   
   <!-- 引入新版Zookeeper依赖包时, 需要解决日志冲突, 剔除日志依赖 -->
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-framework</artifactId>
       <version>2.12.0</version>
   </dependency>
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-recipes</artifactId>
       <version>2.12.0</version>
   </dependency>
   <dependency>
       <groupId>org.apache.zookeeper</groupId>
       <artifactId>zookeeper</artifactId>
       <version>3.4.14</version>
       <!--排除这个slf4j-log4j12-->
       <exclusions>
           <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   
   ```

3. 配置文件`application.properties`中配置`Dubbo`相关属性

   ```properties
   # SpringBoot 启动端口
   server.port=8081
   
   # 当前应用名字
   dubbo.application.name=provider-server
   # 注册中心地址
   dubbo.registry.address=zookeeper://127.0.0.1:2181
   # 扫描指定包下服务
   dubbo.scan.base-packages=com.wyj.service
   
   # dubbo 默认端口是20880,设置端口为-1表示 Dubbo 自动扫描并使用可用端口(从20880开始递增),避免端口冲突的问题
   dubbo.protocol.port=-1
   ```

4. 对服务实现类`TicketServiceImpl`配置服务注解`@Service`

   ```java
   @Service    // 被注解的类可以被扫描到, 在项目一启动就自动注册到注册中心
   @Component  // 使用 Dubbo 后尽量不要用 Service 注解
   public class TicketServiceImpl implements TicketService {
       @Override
       public String getTicket() {
           return "服务商提供卖票服务";
       }
   }
   ```

   > `@Service` 是`org.apache.dubbo.config.annotation.Service`下的注解，启动应用，**Dubbo就会扫描指定的包下带有@Service注解的服务，将它发布在指定的注册中心中**

5. 启动测试，在`dubbo-admin`控制台下观察注册信息：

   ![image-20220107093712151](SpringBoot 基础.assets/image-20220107093712151.png)

   ![image-20220107093754881](SpringBoot 基础.assets/image-20220107093754881.png)

### <a id=_link_3rd>服务消费者</a>

启动服务消费者应用，需要去注册中心订阅服务，代码如下：

1. 导入相关依赖，和服务提供者`provider-server`模块依赖包一样

2. 配置文件`application.properties`中配置`Dubbo`相关属性

   ```properties
   # SpringBoot 启动端口
   server.port=8082
   
   # 当前应用名
   dubbo.application.name=consumer-server
   # 注册中心地址
   dubbo.registry.address=zookeeper://127.0.0.1:2181
   ```

3. 将需要订阅的服务的接口`TicketService`拿过来，要求路径、名字和服务提供者定义的一样

   ![image-20220107111413396](SpringBoot 基础.assets/image-20220107111413396.png)

   > 如果不使用`Dubbo`+`Zookeeper`，实现以上功能需要将服务提供者的接口打包，然后用`pom`文件导入，这里只需要通过注册中心把服务提供者的接口拿过来

4. 编写消费者代码

   ```java
   // 订阅注册中心的服务, 使用买票的服务
   @Service
   public class UserService {
   
       // 想拿到 provider-server 提供票的服务 getTicket(), 要去注册中心拿到服务
       // 使用 @Reference 注解订阅注册中心的服务
       @Reference
       TicketService ticket;
   
       public void buyTicket() {
           String tic = ticket.getTicket();
           System.out.println("到注册中心" + tic);
       }
   
   }
   ```
   
   > `@Service` 是`org.springframework.stereotype.Service`下的注解
   
5. 启动服务提供者`Module`

6. 编写服务消费者测试代码，启动服务消费者`Module`，测试

   ```java
   @SpringBootTest
   class ConsumerServerApplicationTests {
   
       @Autowired
       UserService userService;
   
       @Test
       void contextLoads() {
           userService.buyTicket();
       }
   
   }
   ```

   输出结果：

   <img src="SpringBoot 基础.assets/image-20220108125858662.png" align='left' alt="image-20220108125858662" style="zoom:70%;" />
   
   启动`dubbo-admin`观察情况：
   
   ![image-20220108131207319](SpringBoot 基础.assets/image-20220108131207319.png)
   
   
   
   





# 常见SpringBoot项目错误

## maven打包异常

```text
Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2
```

`pom`文件指定`skipTests`为`true`，跳过单测：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>
                <skipTests>true</skipTests>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <version>3.1.0</version>
        </plugin>  
    </plugins>
</build>
```



































