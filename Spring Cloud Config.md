Spring Cloud Config Server 提供了集中式的配置管理功能，适合在分布式系统中管理多个服务的配置。以下是搭建 Spring Cloud Config Server 的详细步骤，以及如何让服务读取配置中心的配置和动态刷新配置文件。

### 1. 搭建 Spring Cloud Config Server

#### 1.1 添加依赖
创建一个新的 Spring Boot 项目，并在 `pom.xml` 中添加 Spring Cloud Config Server 的依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```

#### 1.2 启用 Config Server
在主类上添加 `@EnableConfigServer` 注解，启用 Spring Cloud Config Server：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

#### 1.3 配置文件
在 `application.yml` 中配置 Config Server 的仓库地址。Config Server 支持多种配置存储方式，常用的存储方式是 Git。

```yaml
server:
  port: 8888 # Config Server 的端口

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-repo/config-repo # Git 仓库地址
          # 配置其他 Git 选项，如认证信息等
```

> **注意**：替换 `https://github.com/your-repo/config-repo` 为实际的配置仓库地址，并确保该仓库中包含配置文件。

#### 1.4 启动 Config Server
启动应用。Config Server 运行后，可以通过 `http://localhost:8888/{application}/{profile}` 获取配置文件内容，例如 `http://localhost:8888/myapp/dev`。

---

### 2. 让服务读取配置中心的配置

为了让微服务从 Config Server 获取配置，需要在每个服务中进行以下配置。

#### 2.1 添加依赖
在 `pom.xml` 中添加 Spring Cloud Config Client 的依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
</dependencies>
```

#### 2.2 配置文件
在微服务的 `bootstrap.yml` 或 `bootstrap.properties` 文件中指定 Config Server 地址，让该服务能够读取配置：

```yaml
spring:
  application:
    name: myapp # 应用名称，对应 Config Server 中配置文件的前缀，如 myapp-dev.yml
  cloud:
    config:
      uri: http://localhost:8888 # Config Server 地址
```

#### 2.3 配置文件结构
在 Config Server 指定的 Git 仓库中创建配置文件，例如 `myapp-dev.yml`，配置内容如下：

```yaml
# myapp-dev.yml 示例内容
server:
  port: 8081

message:
  greeting: "Hello from Config Server!"
```

服务启动时会自动从 Config Server 拉取配置文件 `myapp-dev.yml` 的内容，并将其加载到服务的应用上下文中。

---

### 3. 实现配置文件的动态刷新

为了实现配置的动态刷新，需要借助 Spring Cloud Bus 和 Actuator。Spring Cloud Bus 可以帮助服务间进行消息的传播，从而实现配置的实时刷新。

#### 3.1 添加依赖
在 `pom.xml` 中添加 Spring Cloud Bus 和 Actuator 依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId> <!-- 使用 AMQP（如 RabbitMQ）作为消息中间件 -->
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

> **注意**：Spring Cloud Bus 支持多种消息中间件，比如 RabbitMQ 和 Kafka。上述示例使用 AMQP（RabbitMQ），需要确保 RabbitMQ 已安装并运行。

#### 3.2 配置消息中间件
在 `application.yml` 中配置消息中间件，例如 RabbitMQ：

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

#### 3.3 启用动态刷新
在微服务的主类上添加 `@RefreshScope` 注解，标注需要动态刷新的 Bean。例如：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RefreshScope
@RestController
public class MessageController {

    @Value("${message.greeting}")
    private String greetingMessage;

    @GetMapping("/greeting")
    public String getGreetingMessage() {
        return greetingMessage;
    }
}
```

#### 3.4 手动刷新配置
当 Config Server 中的配置发生变更后，发送 POST 请求到 `/actuator/bus-refresh` 端点触发刷新操作：

```bash
curl -X POST http://localhost:8081/actuator/bus-refresh
```

这个请求会触发 Spring Cloud Bus，在服务集群中广播配置变更消息。所有标注了 `@RefreshScope` 的 Bean 将重新加载配置。

---

通过以上步骤，你可以搭建一个 Spring Cloud Config Server，并在微服务中实现配置中心的读取和动态刷新配置文件。
