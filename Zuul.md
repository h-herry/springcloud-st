Zuul 是由 Netflix 提供的一个 API 网关服务，主要用于动态路由、监控、弹性、权限控制等功能。它可以作为微服务架构中的入口，帮助处理请求并路由到后端服务。以下是一个使用 Zuul 的学习案例。

### 1. 引入依赖

首先，在 Spring Boot 项目中添加 Zuul 的依赖。在 `pom.xml` 中加入以下内容：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

### 2. 启用 Zuul

在 Spring Boot 应用的主类上添加 `@EnableZuulProxy` 注解，以启用 Zuul 网关功能：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}
```

### 3. 配置 Zuul 路由

在 `application.yml` 中配置 Zuul 的路由信息，指定不同的后端服务。例如，假设我们有两个后端服务 `service-a` 和 `service-b`，它们的 Eureka 注册名分别为 `service-a` 和 `service-b`：

```yaml
zuul:
  routes:
    service-a:
      path: /service-a/**
      serviceId: service-a # 指向 service-a 的注册名
    service-b:
      path: /service-b/**
      serviceId: service-b # 指向 service-b 的注册名
```

### 4. 创建后端服务

假设你已经有两个后端微服务 `service-a` 和 `service-b`，它们的简单实现如下：

#### 4.1 `service-a`

创建 `service-a` 服务：

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ServiceAController {

    @GetMapping("/api")
    public String hello() {
        return "Hello from Service A!";
    }
}
```

#### 4.2 `service-b`

创建 `service-b` 服务：

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ServiceBController {

    @GetMapping("/api")
    public String hello() {
        return "Hello from Service B!";
    }
}
```

### 5. 访问服务

在启动 Zuul 网关应用后，可以通过以下 URL 访问后端服务：

- 访问 `service-a`：
  ```
  http://localhost:8080/service-a/api
  ```

- 访问 `service-b`：
  ```
  http://localhost:8080/service-b/api
  ```

### 6. Zuul 过滤器

Zuul 提供了过滤器功能，可以用于请求预处理、后处理、路由等功能。可以创建自定义的 Zuul 过滤器来增强 API 网关的能力。

#### 6.1 创建过滤器

创建一个过滤器类，继承 `ZuulFilter` 类，并实现相应的方法：

```java
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.springframework.stereotype.Component;

@Component
public class CustomFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre"; // 过滤器类型（pre, route, post, error）
    }

    @Override
    public int filterOrder() {
        return 1; // 过滤器顺序
    }

    @Override
    public boolean shouldFilter() {
        return true; // 是否执行该过滤器
    }

    @Override
    public Object run() {
        // 过滤器逻辑
        RequestContext ctx = RequestContext.getCurrentContext();
        String requestURI = ctx.getRequest().getRequestURI();
        System.out.println("Request URI: " + requestURI);
        return null;
    }
}
```

### 7. 使用 Zuul 的动态路由

Zuul 还支持动态路由。你可以在运行时通过配置中心（如 Spring Cloud Config）动态更新路由。

#### 7.1 在配置中心配置

在配置中心中配置路由信息，使用 `spring.cloud.netflix.zuul.routes` 来定义动态路由。例如：

```yaml
spring:
  cloud:
    netflix:
      zuul:
        routes:
          service-c:
            path: /service-c/**
            url: http://localhost:8083 # 指向 service-c 的 URL
```

### 8. 访问监控和管理端点

Zuul 本身并没有内置的监控面板，但可以结合 Spring Boot Actuator 实现监控。可以在 `application.yml` 中配置 Actuator 以监控 Zuul：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

访问 `http://localhost:8080/actuator` 可以看到所有可用的监控和管理端点。

### 总结

通过以上步骤，你可以在 Spring Boot 项目中成功集成 Zuul 作为 API 网关。Zuul 提供了灵活的路由、过滤器机制和监控能力，使得微服务架构中的请求处理更加高效和可靠。
