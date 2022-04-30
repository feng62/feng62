<!DOCTYPE html>
<html>
<head>
<meta  charset=utf-8"/>
    <style>
        mark {
            background-color:#FFFF00 ; font-weight:bold;
        }
    </style>


# WEB项目搭建流程

## 1.application.yml 文件的配置

### 1).mybatis-puls 的配置

- type-aliases-package ：实体类的路径配置
- mapper-locations ：mapper的路径配置

``` yml
mybatis:
  type-aliases-package: com.feng.socket.pojo
  mapper-locations: classpath:mapper/*.xml
```

### 2).spring 的配置

- datasource：数据库据的配置
- redis ：redis数据库的配置
- servlet-multipart：上传文件的配置

``` yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    password: 123456
    url: jdbc:mysql:///reallyshare?useUnicode=true&characterEncoding=utf8
    username: root
  redis:
    host: 127.0.0.1
    password: 123456
    port: 6379
    timeout: 5000
  web:
    resources:
      static-locations: file:D:/ReallyShare/,classpath:/static/
  mvc:
    static-path-pattern: /**
  servlet:
    multipart:
      enabled: true
      max-file-size: 400MB
```



## 2.创建WebConfig文件

创建<mark>WebConfig</mark>文件继承接口<mark id="gao">WebMvcConfigurer</mark> ，然后在类上添加 <mark>@Configuration</mark>注解

###常用的方法

<a href="#addInterceptors"> 拦截器配置</a>  void addInterceptors(InterceptorRegistry registry)
<a href="#addViewControllers">视图跳转控制器</a> void addViewControllers(ViewControllerRegistry registry);
<a href="#addResourceHandlers">静态资源处理 </a>void addResourceHandlers(ResourceHandlerRegistry registry);
<a href="#addCorsMappings"> 解决跨域问题</a>void addCorsMappings(CorsRegistry registry) ;

### <span id="addInterceptors">1).addInterceptors : 拦截器的配置</span>

- addInterceptor：需要一个实现HandlerInterceptor接口的拦截器实例

- addPathPatterns：用于设置拦截器的过滤路径规则；`addPathPatterns("/**")`对所有请求都拦截

- excludePathPatterns：用于设置不需要拦截的过滤规则

- 拦截器主要用途：进行用户登录状态的拦截，日志的拦截等。
``` java
@Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(getmyInterceptor())
                .addPathPatterns("/**")//拦截所有路径
                .excludePathPatterns(new String[]{"/img/**","/ReallyShare/**","/web/**"});
    }
```

### <span id="addViewControllers">2).addViewControllers : 页面跳转</span>

以前写SpringMVC的时候，如果需要访问一个页面，必须要写Controller类，然后再写一个方法跳转到页面，感觉好麻烦，其实重写WebMvcConfigurer中的addViewControllers方法即可达到效果了

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/toLogin").setViewName("login");
}
```

值的指出的是，在这里重写addViewControllers方法，并不会覆盖**WebMvcAutoConfiguration**（Springboot自动配置）中的addViewControllers（在此方法中，Spring Boot将“/”映射至index.html），这也就意味着自己的配置和Spring Boot的自动配置同时有效，这也是我们推荐添加自己的MVC配置的方式。

###<span id="addResourceHandlers">3).addResourceHandlers : 静态资源处理</span>

- addResoureHandler：指的是对外暴露的访问路径

- addResourceLocations：指的是内部文件放置的目录

  可以写多个

```java
 @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
        registry.addResourceHandler("/ReallyShare/**").addResourceLocations("file:D:/ReallyShare/");

    }
```

### <span id="addCorsMappings">4). addCorsMappings：跨域</span>

``` java
@Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                //是否发送Cookie
                .allowCredentials(true)
                //设置放行哪些原始域   SpringBoot2.4.4下低版本使用.allowedOrigins("*")
                .allowedOriginPatterns("*")
                //放行哪些请求方式
                .allowedMethods(new String[]{"GET", "POST", "PUT", "DELETE"})
                .allowedMethods("*") //或者放行全部
                //放行哪些原始请求头部信息
                .allowedHeaders("*")
                //暴露哪些原始请求头部信息
                .exposedHeaders("*");
    }
```

## 3.拦截器

### 1）.拦截器的使用

创建一个继承<mark>HandlerInterceptor</mark> 的类 ,在这个类上实现<mark>@Component</mark>注解

这个类需要在<a href="#addInterceptors">实现接口</a><mark>WebMvcConfigurer</mark>的类中实现<mark>addInterceptors</mark>方法，registry.addInterceptor(new MyInterceptor())添加拦截器

- preHandle :请求处理之前进行调用

  return true 继续请求

  ​			false 结束请求 

  结束请求，自定义响应

  ``` java
  PrintWriter writer = response.getWriter();
          writer.write("");
          writer.flush();
          writer.close();
  ```

- postHandle ：在业务处理器处理请求执行完成后，生成视图之前执行

- afterCompletion ：在DispatcherServlet完全处理完请求后被调用，可用于清理资源等。

``` java
@Component
public class MyInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws IOException {
        PrintWriter writer = response.getWriter();
        writer.write("");
        writer.flush();
        writer.close();
        return false;
//        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```

## 4.MyBatis 的使用

### 1）.安装依赖

``` xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

### 2).在application.yml 中配置

``` java
mybatis:
  type-aliases-package: com.feng.socket.pojo
  mapper-locations: classpath:mapper/*.xml
```

### 3).添加映射器

添加映射器文件，在接口上需要添加<mark>@Mapper</mark>注解

``` java
@Mapper
public interface UserMapper {
    List<User> selectAllUser();
}
```

### 4).添加Mapper.xml

- namespace 用来定义命名空间，该命名空间和定义接口的全限定名一致。

- resultType 表示返回的是一个 Website 类型的值。

- parameterType 表示参数类型

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.feng.mapper.UserMapper">
	<select id="selectAllUser" resultType="user" parameterType="user">
    	select * from user
        <where>
        	<if test="id != null">
            	id = #{id}
            </if>
            <if test="name != null">
                name like "%"#{name} "%"
            </if>
        </where>
         order by rand() LIMIT 30  --随机30条数据
    </select>
</mapper>
```



## 5.上传文件

在spring boot启动类里面添加

```java
@Bean
    public MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        //单个文件最大
        factory.setMaxFileSize(DataSize.parse("400MB")); //KB,MB
        /// 设置总上传数据总大小
        factory.setMaxRequestSize(DataSize.parse("400MB"));
        return factory.createMultipartConfig();
    }
```

## 6. JWT的使用

1.引入依赖

``` xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.18.1</version>
</dependency>
```

jwt创建

- withHeader  :  添加头部（一般使用默认即可）
- withClaim ：创建payload
- withExpiresAt ：定义过期时间，（到具体时间过期）
- sign ：创建签名并设置密钥  Algorithm.HMAC256("")

``` java
//创建token
JWTCreator.Builder builder = JWT.create();
(JWTCreator.Builder)  builder.withHeader(Map<String, Object> headerClaims);
(JWTCreator.Builder)  builder.withClaim(String k,String v);
(JWTCreator.Builder)  builder.withExpiresAt(Date date);
(String)  builder.sign(Algorithm.HMAC256(""));
//解析token
JWT.require(Algorithm.HMAC256("")).build().verify(token);
```

JWT工具类

``` java
public class JWTutils {
    
    private static final String SING = "RTYHFG%^*$$jjbujyg%^&^&*u";
    
    /**
     * 生成token
     */
    public  static String getToken(Map<String , String> map){
        Calendar instance = Calendar.getInstance();
        instance.add(Calendar.DATE ,30);//默认30天过期
        JWTCreator.Builder builder = JWT.create();
        map.forEach((k,v)->{
            builder.withClaim(k,v);
        });
        String token = builder.withExpiresAt(instance.getTime()) //指定令牌时间
                .sign(Algorithm.HMAC256( SING ));
        return token;
    }
    
    /**
     * 验证token
     */
    public static DecodedJWT verify (String token){
        return JWT.require(Algorithm.HMAC256( SING )).build().verify( token );
    }

}
```

JWT的异常判断,可以将其添加到拦截器中

``` java
Map<String , Object> map = new HashMap<>();
        String token = request.getHeader("token");
        try {
            JWTutils.verify(token);
        }catch (SignatureVerificationException e ){
            e.printStackTrace();
            map.put("message","无效签名");
        }catch (TokenExpiredException e ){
            e.printStackTrace();
            map.put("message","token过期");
        }catch (AlgorithmMismatchException e ){
            e.printStackTrace();
            map.put("message","token算法不一致");
        }catch (Exception e ){
            e.printStackTrace();
            map.put("message","token无效");
        }
```

