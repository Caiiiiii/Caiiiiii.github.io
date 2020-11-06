---
layout:     post
title:      "JWT 与 security 的结合"
subtitle:   "Hello World, Hello Blog"
date:       2020-10-23
author:     "Caiiiiii"
header-img: "img/292030_screenshots_20201022183122_1.jpg"
tags:
    - Java
    - Spring Security
    - JWT
---

# JWT + Spring Security
本文讲解的是Mall项目中JWT和Spring Security的搭建过程
本文基本参考搭建  https://github.com/echisan/springboot-jwt-demo/blob/master/blog_content.md
## 简介
JWT 与 Spring Security的结合，是一个基于token的权限管理系统。这个方案利于前后端分离的开发工作。

## 什么是JWT
JSON WEB Token（JWT），是一种基于JSON的、用于在网络上声明某种主张的令牌（token）。JWT通常由三部分组成: 头信息（header）, 消息体（payload）和签名（signature）。

### 用途
- 授权：这是JWT的最常见使用场景。 一旦用户登录，后续每个请求将携带JWT，从而允许用户访问该token允许的路径，服务和资源。 单一登录是当今广泛使用JWT的一项功能，因为它的开销很小并且可以在不同的域名中轻松使用。
- 信息交换：JWT是在各方之间安全传输信息的一种好方法。 因为可以对JWT进行签名（例如，使用公钥/私钥对），所以您可以确定发送人是他们自己。 此外，由于签名是使用header和payload计算的，因此您还可以验证内容是否被篡改。

### 结构
头信息指定了该JWT使用的签名算法:

header = '{"alg":"HS256","typ":"JWT"}'
HS256 表示使用了 HMAC-SHA256 来生成签名。

消息体包含了JWT的意图：

payload = '{"loggedInAs":"admin","iat":1422779638}'//iat表示令牌生成的时间
未签名的令牌由base64url编码的头信息和消息体拼接而成（使用"."分隔），签名则通过私有的key计算而成：

key = 'secretkey'  
unsignedToken = encodeBase64(header) + '.' + encodeBase64(payload)  
signature = HMAC-SHA256(key, unsignedToken) 
最后在未签名的令牌尾部拼接上base64url编码的签名（同样使用"."分隔）就是JWT了：

```
token = encodeBase64(header) + '.' + encodeBase64(payload) + '.' + encodeBase64(signature) 

token看起来像这样: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsb2dnZWRJbkFzIjoiYWRtaW4iLCJpYXQiOjE0MjI3Nzk2Mzh9.gzSraSYS8EXBxLN_oWnFSRgCzcmJmMjLiuyu5CSpyHI 
```
JWT常常被用作保护服务端的资源（resource），客户端通常将JWT通过HTTP的Authorization header发送给服务端，服务端使用自己保存的key计算、验证签名以判断该JWT是否可信：

Authorization: Bearer eyJhbGci*...<snip>...*yu5CSpyHI

ps:![参考文章](https://zhuanlan.zhihu.com/p/99705304)

### 总结
基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或会话信息。这也就意味着机遇tokent认证机制的应用不需要去考虑用户在哪一台服务器登陆了，这就为应用的扩展提供了便利。

## 什么是Spring Security
Spring Security是为基于Spring的应用程序提供声明式安全保护的安全性框架，它能够在web请求级别和方法调用级别处理身份认证和授权。由于是基于Spring框架，它充分地利用了依赖注入和面向切面技术。Spring Security从两个角度来解决安全性问题，它使用Servlet规范中的Filter保护web请求并限制URL级别的访问，通过Spring AOP保护方法调用。

## 搭建流程

### 在项目中引入JWT的Maven
```
      <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
         <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.0</version>
        </dependency>
```

### JwtTokenUtils
JWT工具类。对JJWT封装:创建token,从token中获取信息等。
```
public class JwtTokenUtils {

    public static final String TOKEN_HEADER = "Authorization";
    public static final String TOKEN_PREFIX = "Bearer";

    //secret就是用来进行jwt的签发和jwt的验证
    private static final String SECRET = "JWTSECRET";

    //JWT 签发者
    private static final String ISS = "mall";

    // 过期时间是3600秒，既是1个小时
    private static final long EXPIRATION = 3600L;

    // 选择了记住我之后的过期时间为7天
    private static final long EXPIRATION_REMEMBER = 604800L;

    // 角色的key
    private static final String ROLE_CLAIMS = "rol";

    // 创建token
    public static String createToken(String username,String role, boolean isRememberMe) {
        long expiration = isRememberMe ? EXPIRATION_REMEMBER : EXPIRATION;
        HashMap<String, Object> map = new HashMap<>();
        map.put(ROLE_CLAIMS, role);
        return Jwts.builder()
                .signWith(SignatureAlgorithm.HS512, SECRET)
                .setClaims(map)
                .setIssuer(ISS)
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + expiration * 1000))
                .compact();
    }

    // 从token中获取用户名
    public static String getUsername(String token){
        return getTokenBody(token).getSubject();
    }

    // 从token中获取权限
    public static String getUserRole(String token){
        return (String) getTokenBody(token).get(ROLE_CLAIMS);
    }

    // 是否已过期
    public static boolean isExpiration(String token){
        return getTokenBody(token).getExpiration().before(new Date());
    }

    private static Claims getTokenBody(String token){
        return Jwts.parser()
                .setSigningKey(SECRET)
                .parseClaimsJws(token)
                .getBody();
    }
}
```

### 编写一个根据用户名获取用户的方法
```
    @Override
    public AdminLogin findAdminByUserName(String username) {
        return adminLoginMapper.findAdminByUserName(username);
    }
```

### 创建JwtUser对象类实现UserDetails接口。
```
public class JwtUser implements UserDetails {

    private Integer id;
    private String username;
    private String password;
    private Collection<? extends GrantedAuthority> authorities;

    public JwtUser() {
    }

    // 写一个能直接使用user创建jwtUser的构造器
    public JwtUser(AdminLogin user, RoleInfo role) {
        id = user.getAdminId();
        username = user.getLoginName();
        password = user.getPassword();
       if(role != null){
           authorities = Collections.singleton(new SimpleGrantedAuthority(role.getName()));
       }else{
           authorities = Collections.singleton(new SimpleGrantedAuthority("ROLE_NORMAL"));
       }
    }

    //
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    // 账号是否未过期，默认是false，记得要改一下
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    // 账号是否未锁定，默认是false，记得也要改一下
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    // 账号凭证是否未过期，默认是false，记得还要改一下
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    // 这个有点抽象不会翻译，默认也是false，记得改一下
    @Override
    public boolean isEnabled() {
        return true;
    }

    // 我自己重写打印下信息看的
    @Override
    public String toString() {
        return "JwtUser{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", authorities=" + authorities +
                '}';
    }

```

### CustomUserDetailsServiceImpl
使用springSecurity需要实现UserDetailsService接口供权限框架调用，根据用户名去获取用户,再通过用户查找到其角色权限。并创建返回JwtUser
```
@Service
public class CustomUserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private AdminService adminService;

    @Autowired
    private RoleService roleService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 根据用户名验证用户
        AdminLogin adminLogin = adminService.findAdminByUserName(username);
        if (adminLogin == null) {
            throw new UsernameNotFoundException("用户名不存在");
        }

      RoleInfo roleInfo = adminService.getRoleById(adminLogin.getAdminId());
        return new JwtUser(adminLogin,roleInfo);
    }
}
```

### 配置拦截器
需要重写两个方法，一个是用户验证，一个是权限验证。
使用JWTAuthenticationFilter去进行用户账号的验证，使用JWTAuthorizationFilter去进行用户权限的验证。

### JWTAuthenticationFilter

JWTAuthenticationFilter继承于UsernamePasswordAuthenticationFilter 该拦截器用于获取用户登录的信息，只需创建一个token并调用authenticationManager.authenticate()让spring-security去进行验证就可以了，不用自己查数据库再对比密码了，这一步交给spring去操作。 
```
//用户验证
public class JWTAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private AuthenticationManager authenticationManager;

    public JWTAuthenticationFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
        super.setFilterProcessesUrl("/admin/login");
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {

        // 从输入流中获取到登录的信息
        try {
            AdminLogin loginUser = new ObjectMapper().readValue(request.getInputStream(), AdminLogin.class);
            return authenticationManager.authenticate(
                    new UsernamePasswordAuthenticationToken(loginUser.getLoginName(), loginUser.getPassword(), new ArrayList<>())
            );
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    // 成功验证后调用的方法
    // 如果验证成功，就生成token并返回
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {

        // 查看源代码会发现调用getPrincipal()方法会返回一个实现了`UserDetails`接口的对象
        // 所以就是JwtUser啦
        JwtUser jwtUser = (JwtUser) authResult.getPrincipal();
        System.out.println("jwtUser:" + jwtUser.toString());

        String role = "";
        Collection<? extends GrantedAuthority> authorities = jwtUser.getAuthorities();
        for (GrantedAuthority authority : authorities){
            role = authority.getAuthority();
        }

        String token = JwtTokenUtils.createToken(jwtUser.getUsername(),role, false);
        // 返回创建成功的token
        // 但是这里创建的token只是单纯的token
        // 按照jwt的规定，最后请求的格式应该是 `Bearer token`
        response.setHeader("token", JwtTokenUtils.TOKEN_PREFIX + token);
    }

    // 这是验证失败时候调用的方法
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        response.getWriter().write("authentication failed, reason: " + failed.getMessage());
    }
}

```

### JWTAuthorizationFilter
验证成功当然就是进行鉴权了，每一次需要权限的请求都需要检查该用户是否有该权限去操作该资源，当然这也是框架帮我们做的，那么我们需要做什么呢？很简单，只要告诉spring-security该用户是否已登录，是什么角色，拥有什么权限就可以了。 JWTAuthenticationFilter继承于BasicAuthenticationFilter。只要确保过滤器在config里的顺序，JWTAuthorizationFilter在JWTAuthenticationFilter后面就没问题了。
```
public class JWTAuthorizationFilter extends BasicAuthenticationFilter {

    public JWTAuthorizationFilter(AuthenticationManager authenticationManager) {
        super(authenticationManager);
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws IOException, ServletException {

        String tokenHeader = request.getHeader(JwtTokenUtils.TOKEN_HEADER);
        // 如果请求头中没有Authorization信息则直接放行了
        if (tokenHeader == null || !tokenHeader.startsWith(JwtTokenUtils.TOKEN_PREFIX)) {
            chain.doFilter(request, response);
            return;
        }
        // 如果请求头中有token，则进行解析，并且设置认证信息
        SecurityContextHolder.getContext().setAuthentication(getAuthentication(tokenHeader));
        super.doFilterInternal(request, response, chain);
    }

    // 这里从token中获取用户信息并新建一个token
    private UsernamePasswordAuthenticationToken getAuthentication(String tokenHeader) {
        String token = tokenHeader.replace(JwtTokenUtils.TOKEN_PREFIX, "");
        String username = JwtTokenUtils.getUsername(token);
        String role = JwtTokenUtils.getUserRole(token);
        if (username != null){
            return new UsernamePasswordAuthenticationToken(username, null,
                    Collections.singleton(new SimpleGrantedAuthority(role))
            );
        }
        return null;
    }

```

### 统一结果处理
统一处理被403响应的事件，新建JWTAuthenticationEntryPoint类
```
public class JWTAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {

        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json; charset=utf-8");
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        String reason = "统一处理，原因："+authException.getMessage();
        response.getWriter().write(new ObjectMapper().writeValueAsString(reason));
    }
}
```


### 配置SpringSecurity
需要开启一下注解@EnableWebSecurity然后再继承一下WebSecurityConfigurerAdapter.
```
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    String[] SWAGGER_WHITELIST = {
            "/swagger-ui.html",
            "/swagger-ui/**",
            "/swagger-resources/**",
            "/v2/api-docs",
            "/v3/api-docs",
            "/webjars/**"
    };

    @Autowired
    // 因为UserDetailsService的实现类实在太多啦，这里设置一下我们要注入的实现类
    @Qualifier("customUserDetailsServiceImpl")
    private UserDetailsService userDetailsService;

    // 加密密码的，安全第一嘛~
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()
                .antMatchers(SWAGGER_WHITELIST)
                .antMatchers("/admin/login") ;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable()
                .authorizeRequests()
                // 测试用资源，需要验证了的用户才能访问
                .antMatchers("/admin/**").authenticated()
//                .antMatchers("/admin/login").permitAll()
//                // 其他都放行了
//                .anyRequest().permitAll()
                .and()
                .addFilter(new JWTAuthenticationFilter(authenticationManager()))
                .addFilter(new JWTAuthorizationFilter(authenticationManager()))
                // 不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
                .and()
                .exceptionHandling().authenticationEntryPoint(new JWTAuthenticationEntryPoint());
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource() {
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", new CorsConfiguration().applyPermitDefaultValues());
        return source;
    }

```

### 到目前为止就可以进行测试登录了。当然前提先编写一个注册方法。
那登录在哪呢？我们看一下UsernamePasswordAuthenticationFilter的源代码
```
public UsernamePasswordAuthenticationFilter() {
		super(new AntPathRequestMatcher("/login", "POST"));
	}
```
可以看出来默认是/login，所以登录直接使用这个路径就可以了。当然也可以自定义 只需要在JWTAuthenticationFilter的构造方法中加入下面那一句话就可以。
```
 public JWTAuthenticationFilter(AuthenticationManager authenticationManager) {
        this.authenticationManager = authenticationManager;
        super.setFilterProcessesUrl("/auth/login");
    }
```

### 测试
可以看到postman中，在登录成功后的响应头部会有一个生成的token。
接下来只需要把该响应头添加到我们的请求头上去，这里需要把Bearer[空格]去掉，注意Bearer后的空格也要去掉，因为postman再选了BearerToken之后会自动在token前面再加一个Bearer。
![测试图片](/img/postmanbyjwtlogin.png)

### 配置访问权限
在权限配置方面，有2种方法。并且有2种写法。
方法1：在springSecurity中配置访问权限
```
意思是只有角色为ROLE_ADMIN才有权限删除该资源
写法1：
.antMatchers(HttpMethod.DELETE, "/tasks/**").hasAuthority("ROLE_ADMIN")
写法2：
*使用了ROLE_作为前缀前提下*
.antMatchers(HttpMethod.DELETE, "/tasks/**").hasRole("ADMIN")
```

方法2：注解中配置访问权限
```
在SecurityConfig中配置的注解
@EnableGlobalMethodSecurity(prePostEnabled = true)
写法1：
 @PostMapping
    @PreAuthorize("hasAuthority('ROLE_ADMIN')")
    public String newTasks(){
        return "创建了一个新的任务";
    }


写法2：
在方法中使用注解 *使用了ROLE_作为前缀前提下*
  @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public String newTasks(){
        return "创建了一个新的任务";
    }

```

