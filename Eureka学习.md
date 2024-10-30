# 1. 搭建 Eureka Server

首先，我们需要创建一个 Spring Boot 项目并添加 `Eureka Server` 依赖。Eureka Server 将作为服务注册中心，让其他微服务可以注册到它上面。

## 1.1 添加依赖
在 `pom.xml` 文件中添加 `spring-cloud-starter-netflix-eureka-server` 依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

### 1.2 启用 Eureka Server
在主类上添加 `@EnableEurekaServer` 注解，启用 Eureka Server：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### 1.3 配置文件
在 `application.yml` 文件中进行基础配置：

```yaml
server:
  port: 8761 # Eureka Server 的默认端口

eureka:
  client:
    register-with-eureka: false # 当前服务本身不注册到 Eureka
    fetch-registry: false # 当前服务不从 Eureka 拉取服务列表
```

### 1.4 启动 Eureka Server
启动应用，访问 `http://localhost:8761`，可以看到 Eureka Server 的注册页面。

---

## 2. 注册 Eureka Client

Eureka Client 是实际的微服务应用，它会注册到 Eureka Server 上，让其他服务可以发现它。  

### 2.1 添加依赖
在 `pom.xml` 中添加 `spring-cloud-starter-netflix-eureka-client` 依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

### 2.2 启用 Eureka Client
在微服务的主类中添加 `@EnableEurekaClient` 注解：

**高版本不需要添加该注解**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

### 2.3 配置文件
在 `application.yml` 文件中配置 Eureka Server 的地址，使该服务能够注册到 Eureka：

```yaml
spring:
  application:
    name: eureka-client # 微服务名称

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka # Eureka Server 地址
```

### 2.4 启动 Eureka Client
启动微服务应用，观察 Eureka Server 页面，可以看到新的服务实例已注册。

---

## 3. 实现负载均衡

在 Spring Cloud 中，Eureka 与 Ribbon 可以实现客户端的负载均衡。Ribbon 会从 Eureka 注册中心获取服务实例列表，并自动选择一个实例进行调用。

### 3.1 添加 Ribbon 依赖
Ribbon 已经集成在 `spring-cloud-starter-netflix-eureka-client` 中，因此不需要额外依赖。只需在调用服务时使用 `RestTemplate` 配合负载均衡注解即可。

### 3.2 配置 RestTemplate
在服务消费者中配置 `RestTemplate`，并启用负载均衡：

```java
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### 3.3 调用其他服务
在代码中调用其他服务时，通过服务名称而非具体的 IP 地址：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class MyController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/callService")
    public String callService() {
        // 使用 Eureka 的服务名称调用
        String serviceUrl = "http://eureka-client/service";
        return restTemplate.getForObject(serviceUrl, String.class);
    }
}
```

通过上述配置，`RestTemplate` 会自动选择一个可用的 `eureka-client` 实例进行调用，从而实现负载均衡。
