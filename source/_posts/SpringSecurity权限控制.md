---
title: SpringSecurity权限控制
date: 2020-11-09 21:47:57
tags: [Spring,SpringSecurity]
categories: [Java,Spring]
---

## spring security 简介

Spring Security是一个功能强大且高度可定制的身份验证和访问控制框架。它是用于保护基于Spring的应用程序的实际标准。

Spring Security是一个框架，致力于为Java应用程序提供身份验证和授权。与所有Spring项目一样，Spring Security的真正强大之处在于可以轻松扩展以满足自定义要求

<!--more-->

## 执行流程

![20201109124625](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202205192027289.png)

![20201109163442](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202205192027332.png)

## 基本原理

SpringSecurity本质是一个过滤器

- FilterSecurityInterceptor: 是一个方法级的权限过滤器,基本位于过滤链的最底部
- ExceptionTranslationFilter: 是个异常过滤器，用来处理在认证授权过程中抛出的异常
- UsernamePasswordAuthenticationFilter: 对/ login的POST请求做拦截，校验表单中用户名，密码。
  - attemptAuthentication: 处理用户名密码
  - successfulAuthenticaiton: 认证成功处理逻辑
  - unsuccessfulAuthenticaiton: 认证失败处理逻辑


## 重要接口

- UserDetailsService接口: 查询数据库用户名和密码过程, 创建类实现UserDetailService，编写查询数据过程，返回User对象这个User对象是安全框架提供对象

  ```java
  @Component
  public class MyUserDetailsService implements UserDetailsService {
      @Override
      public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
          //1.数据库查询admin对象
          Admin admin = AdminMapper.select....byName();
          //2.给admin设置角色权限信息
          List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
          authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
          authorities.add(new SimpleGrantedAuthority("UPDATE"));
          //3.把admin对象和authorities封装到UserDetails中
          String password = admin.getPassword();
          return new User(username,password,authorities);
      }
  }
  ```

  *注意:SpringSecurity会在角色字符串前面加"ROLE"前缀,从数据库查询得到的用户信息,角色信息,权限信息需要之际手动组装,组装时同样要在角色字符串前面加"ROLE_"*

- PasswordEncoder接口: 数据加密接口，用于返回User对象里面密码加密

## Web权限方案

### 配置类使用

1. 继承WebSecurityConfigurerAdapter 并重写configure方法,并加入到IOC容器

   - @EnableWebMvcSecurity注解：在Spring 4.0中已弃用。

   - WebSecurityConfigurerAdapter类：可以通过重载该类的三个configure()方法来制定Web安全的细节。(过时)
   - 使用 `SecurityFilterChain` @Bean 来配置 `HttpSecurity`
   - 使用 `WebSecurityCustomizer` Bean 来配置 `WebSecurity`

   

   ```java
   @Configuration
   @EnableWebSecurity
   public class WebAppSecurity extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(HttpSecurity http) throws Exception {
   		//通过重载该方法，可配置如何通过拦截器保护请求。
       }
   
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
   		//构建User对象==>userDetailsService (用户名密码) 
           auth.userDetailsService(userDetailServiceImpl).passwordEncoder(getPasswordEncoder());
       }
   
       @Override
       public void configure(WebSecurity web) throws Exception {
   		//通过重载该方法，可配置Spring Security的Filter链
           web.ignoring().antMatchers("/static")
       }
   }
   ```

2. HttpSecurity http

   ```java
       @Override
     	protected void configure(HttpSecurity http) throws Exception {
           http
               //修改Spring Security默认的登陆界面
               .formLogin()
               .loginPage("/to/login/page.html")
               .loginProcessingUrl("/do/login.html")
               .defaultSuccessUrl("/to/main/page.html	")
               .permitAll()			//无条件访问
               
               .authorizeRequests()	//对请求进行授权 
               //访问"/"和"/home"路径的请求都允许
               .antMatchers("/","/home")
               .permitAll()
               
               //而其他的请求都需要认证
               .anyRequest()
               .authenticated()
               .and()
               .csrf().disable() //关闭csrf防护
   
               //对url设置访问要求
               .antMathers("/test/index")		
               .hasRole()				//要求用户具备的角色
               .usernameParameter("loginAcct")
               .passwordParameter("userPwd")
               .and()
               .logout()	
               .logoutUrl("/do/logout.html")
               
               //指定异常处理器
               .exceptionHandling()	
   //            .accessDeniedPage("/to/error/page.html")	//访问被拒时去的页面
               .accessDeniedHandler(new AccessDeniedHandler()) //定制异常处理
               
               //记住登录状态
   			.rememberMe()
               //setCreateTableOnStartup 自动创建表
               .tokenRepository(@Bean new PersistentTokenRepository.setDataSourse())
               .tokenValiditySeconds(60)
               .userDetailsService(userDetailServiceImpl)
   	
       }
   ```
   
3. 登录信息保存(Remenber me)

   ![image-20220908142444889](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/image-20220908142444889.png)

   ```html
   记住我:<input type="checkbox" name="remember-me" title=记住密码"/>
   ```

   

| 方法                     | 作用                                                         |
| ------------------------ | :----------------------------------------------------------- |
| access(String)           | 如果给定的SpEL表达式计算结果为true，就允许访问               |
| anonymous()              | 允许匿名用户访问                                             |
| authenticated()          | 允许认证过的用户访问                                         |
| denyAll()                | 无条件拒绝所有访问                                           |
| fullyAuthenticated()     | 如果用户是完整认证的话（不是通过Remember-me功能认证的），就允许访问 |
| hasAnyAuthority(String…) | 如果用户具备给定权限中的某一个的话，就允许访问               |
| hasAnyRole(String…)      | 如果用户具备给定角色中的某一个的话，就允许访问               |
| **hasAuthority(String)** | **如果用户具备给定权限的话，就允许访问**                     |
| hasIpAddress(String)     | 如果请求来自给定IP地址的话，就允许访问                       |
| **hasRole(String)**      | **如果用户具备给定角色的话，就允许访问**                     |
| not()                    | 对其他访问方法的结果求反                                     |
| permitAll()              | 无条件允许访问                                               |
| rememberMe()             | 如果用户是通过Remember-me功能认证的，就允许访问              |
| usernameParameter()      | 定制登录账号请求参数名                                       |
| passwordParameter()      | 定制登录密码请求参数名                                       |
| defaultSuccessUrl()      | 登录成功后去往的页面                                         |
| loginProcessingUrl()     | 登录放行                                                     |



(1).引入SpringSecurity标签库

```html
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```

(2).获取(UsernamePasswordAuthenticationToken会擦除密码)

```html
<security:authentication property="principal.original.username" />
```

### 注解使用

```java
@Configuration
@EnableWebSecurity
    @EnableGlobalMethodSecurity(prePostEnabled = true)	//开启注解 (配置类或者启动类上)
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure........
}
```

Controller上使用注解:@PreAuthority,@PostAuthority,@PreFilter,@PostFilter

- @Secured: 具有某个角色可以访问
- @PreAuthority: 进入方法前, 验证权限
- @PostAuthority: 进入方法后, 验证权限
- @PreFilter: 对传入数据做过滤
- @PostFilter: 对返回数据做过滤

### **csrf**

用来生成token防止跨站请求伪造,需要在表单添加

```html
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}">
```

或者配置HttpSecurity禁用否则会发生403错误

```java
http.csrf().disable();
```





### 配置文件使用

**配置xml**

SpringSecurity使用的是过滤器Filter而不是拦截器Interceptor,意味着SpringSecurity能够管理的不仅仅是SpringMVC中的handler请求,还包含Web应用中的所有请求,包括静态资源

```xml
  <filter>
    <filter-name>springSecurityFileterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFileterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

filter-name中的名字必须是springSecurityFileterChain才能加载到ioc容器中的Filter

**导入依赖**

```xml
<!-- SpringSecurity 对 Web 应用进行权限管理 -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
</dependency>
<!-- SpringSecurity 配置 -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
<!-- SpringSecurity 标签库 -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
</dependency>
```

### 



### 遇到的问题

> No bean named 'springSecurityFilterChain' available:

当SpringSecurity配置类添加到SpringMvc Ioc中时,会抛出找不到pspringSecurityFilterChain异常

- 三大组件启动顺序:

  首先:ContextLoaderListener初始化,创建Spring的IOC容器

  其次:DelegatingFilterProxy初始化,查找IOC容器,查找bean

  最后:DispatherServlet初始化,创建SpringMVC的IOC容器

- Filter查找IOC容器然后找Bean的工作机制

![20201110111808](https://lzc-oss.oss-cn-chengdu.aliyuncs.com/notes/202205192028931.png)

- 解决方案一:把两个ioc容器合二为一

  不使用ContextLoaderListener,让Dispater加载所有Spring配置文件,但是会破环现有程序结构

- 解决方案二:改源码

  修改DelegatingFilterProxy类的initFilterBean(),doFilter()方法

```java
    @Override
    protected void initFilterBean() throws ServletException {
        synchronized (this.delegateMonitor) {
            if (this.delegate == null) {
                // If no target bean name specified, use filter name.
                if (this.targetBeanName == null) {
                    this.targetBeanName = getFilterName();
                }
                // Fetch Spring root application context and initialize the delegate early,
                // if possible. If the root application context will be started after this
                // filter proxy, we'll have to resort to lazy initialization.

                /*注释掉
                    WebApplicationContext wac = findWebApplicationContext();
                    if (wac != null) {
                        this.delegate = initDelegate(wac);
                    }*/
            }
        }
    }
```

```java
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		// Lazily initialize the delegate if necessary.
		Filter delegateToUse = this.delegate;
		if (delegateToUse == null) {
			synchronized (this.delegateMonitor) {
				delegateToUse = this.delegate;
				if (delegateToUse == null) {
                    //把原来查找IOC容器的代码注释掉,按需要重新编写
					//WebApplicationContext wac = findWebApplicationContext();
                    //1.获取ServletContext对象
                    ServletContext sc = this.getServletContext();
                    //2.拼接SpingMvc将Ioc容器存入ServletContext域的时候使用的属性名
                    String servletName = "springDispatherServlet";
                    String attrName = FrameworkServlet.SERVLET_CONTEXT_PREFIX+servletName;
                    //3.根据attrName从ServletCOntext域中获取IOC容器对象
                    WebApplicationContext wac = (WebApplicationContext)sc.getAttribute(attrName)
					if (wac == null) {
						throw new IllegalStateException("No WebApplicationContext found: " +
								"no ContextLoaderListener or DispatcherServlet registered?");
					}
					delegateToUse = initDelegate(wac);
				}
				this.delegate = delegateToUse;
			}
		}
```



> Cannot pass a pull GrantedAuthorrity collection:

没有设置roles()或者authorities方法导致的

> Spring Security 登陆报错：There is no PasswordEncoder mapped for the id “null”:
>
> 必须设置密码的加密方式

5.0以后对于密码的管理有些变化,现如今Spring Security中密码的存储格式是“{id}…………”。前面的id是加密方式，id可以是bcrypt、sha256等，后面跟着的是加密后的密码。也就是说，程序拿到传过来的密码的时候，会首先查找被“{”和“}”包括起来的id，来确定后面的密码是被怎么样加密的，如果找不到就认为id是null。

```java
auth.passwordEncoder(new BCryptPasswordEncoder())
    .password(new BCryptPasswordEncoder().encode("123123"))
```

> @PreAuthorize()无法拦截：

在使用@PreAuthorize()做拦截时,如果同时也设置了@EnableGlobalMethodSecurity(prePostEnabled = true),还是无法拦截,可能时因为在扫描SpringSecurity时是使用Spring扫描,而拦截注解加在了请求上,将注解加在Service文件或者是Mapper文件即可,反之亦然

## 微服务权限方案

1. 继承UsernamePasswordAuthenticationFilter类, 重写attemptAuthentication()认证逻辑, (保存token储存的权限信息)

   ```java
   public class LoginFilter extends UsernamePasswordAuthenticationFilter {
   
       private AuthenticationManager authenticationManager;
       private TokenManager tokenManager;
       public LoginFilter(AuthenticationManager authenticationManager, TokenManager tokenManager){
           this.authenticationManager = authenticationManager;
           this.tokenManager = tokenManager;
       }
   
   
       //认证token
       @Override
       public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
           User user = new User(request.getParameter("username"), request.getParameter("password"), new ArrayList<>());
               return authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(user.getUsername(),user.getPassword(),user.getAuthorities()));
       }
   
       //认证成功 生成token
       @Override
       protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
           User user = (User) authResult.getPrincipal();
           String token = tokenManager.createToken(user.getUsername());
   		...保存token...
       }
   
       //认证失败
       @Override
       protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
           System.err.println("认证失败!");
       }
   }
   ```

2. 继承BasicAuthenticationFilter类, 重写doFilterInternal过滤逻辑 , (监控token)

   ```java
   public class AuthFilter extends BasicAuthenticationFilter {
   
       private TokenManager tokenManager;
       public AuthFilter(AuthenticationManager authenticationManager, TokenManager tokenManager) {
           super(authenticationManager);
           this.tokenManager = tokenManager;
       }
   
       @Override
       protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
           //获取当前认证成功用户权限信息
           UsernamePasswordAuthenticationToken authRequest = getAuthentication(request);
           //判断如果有权限信息，放到权限上下文中
           if(authRequest != null) {
               SecurityContextHolder.getContext().setAuthentication(authRequest);
           }
           chain.doFilter(request,response);
       }
   
       private UsernamePasswordAuthenticationToken getAuthentication(HttpServletRequest request) {
           //从header获取token
           String token = request.getHeader("token");
           if(token != null) {
               //从token获取用户名
               String username = tokenManager.getUserInfoFromToken(token);
               //从redis获取对应权限列表
               List<String> permissionValueList = getTokenByUsername();
               Collection<GrantedAuthority> authority = new ArrayList<>();
               for(String permissionValue : permissionValueList) {
                   SimpleGrantedAuthority auth = new SimpleGrantedAuthority(permissionValue);
                   authority.add(auth);
               }
               return new UsernamePasswordAuthenticationToken(username,token,authority);
           }
           return null;
       }
   
   }
   ```
