---
title: "Spring Boot 3集成Redis"
date: 2023-03-19T21:36:18+08:00
tags: ["Spring Boot 3", "Redis"]
---

Spring Boot 3真的很香，最大的改变可能就是GraalVM的支持了，直接可以打包成可以运行程序，把jre直接省掉了，性能得到了很好的提升。

但是升级完成后，集成Redis的时候直接傻眼了，一直报错，我们来看一下堆栈信息：

```bash
org.springframework.data.redis.RedisConnectionFailureException: Unable to connect to Redis
Caused by: io.lettuce.core.RedisConnectionException: Unable to connect to localhost/<unresolved>:6379
Caused by: io.netty.channel.AbstractChannel$AnnotatedConnectException: Connection refused: localhost/127.0.0.1:6379
Caused by: java.net.ConnectException: Connection refused
```

<!-- more -->

再来看一下配置文件 `application.yml`

```bash
spring:
  redis:
    host: redis
    port: 6379
    password:
```

这里解释一下 host 设置为 redis 的原因，我是在 docker-compose.yml 配置的service和redis服务，在本地使用的 127.0.0.1 好好的，心想到线上改成docker compose容器的hostname就可以了。结果死活报连接出错。

## 推测

首先，推测是不是redis.conf没有配置允许所有外部机器连接。把redis.conf配置好之后，依然报错。

然后，推测是不是配置文件没有正确的映射到docker容器里面，进入到容器进行检查，配置没有问题。

最后，推测是不是spring data redis的默认`LettuceConnectionFactory` 连接没有读取到配置文件的内容，然后进行一番调试，发现如果不创建一个新的`LettuceConnectionFactory` 那么默认就会连接到 `localhost:6379` 上去。

## 调试

先找到类`LettuceConnectionFactory` 在所有的构造函数里面打上断点，调试一次后，发现还有一个`LettuceConnectionConfiguration`在前面进行配置的创建，方法如下：

```java
class LettuceConnectionConfiguration extends RedisConnectionConfiguration {
  private LettuceConnectionFactory createLettuceConnectionFactory(LettuceClientConfiguration clientConfiguration) {
    if (getSentinelConfig() != null) {
      return new LettuceConnectionFactory(getSentinelConfig(), clientConfiguration);
    }
    if (getClusterConfiguration() != null) {
      return new LettuceConnectionFactory(getClusterConfiguration(), clientConfiguration);
    }
    return new LettuceConnectionFactory(getStandaloneConfig(), clientConfiguration);
  }
}
```

由于没有cluster配置，运行到了`return new LettuceConnectionFactory(getStandaloneConfig(), clientConfiguration);`这一行。

```java
protected final RedisStandaloneConfiguration getStandaloneConfig() {
  if (this.standaloneConfiguration != null) {
    return this.standaloneConfiguration;
  }
  RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
  if (StringUtils.hasText(this.properties.getUrl())) {
    ConnectionInfo connectionInfo = parseUrl(this.properties.getUrl());
    config.setHostName(connectionInfo.getHostName());
    config.setPort(connectionInfo.getPort());
    config.setUsername(connectionInfo.getUsername());
    config.setPassword(RedisPassword.of(connectionInfo.getPassword()));
  }
  else {
    config.setHostName(this.properties.getHost());
    config.setPort(this.properties.getPort());
    config.setUsername(this.properties.getUsername());
    config.setPassword(RedisPassword.of(this.properties.getPassword()));
  }
  config.setDatabase(this.properties.getDatabase());
  return config;
}
```

在 `RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();` 这一行创建了一个默认的配置。

我们继续进行类`RedisStandaloneConfiguration`进行查看，上面调用得是一个无参的构造方法，属性都是使用的默认值，找到属性查看

```java
private static final String DEFAULT_HOST = "localhost";
private static final int DEFAULT_PORT = 6379;

private String hostName = DEFAULT_HOST;
private int port = DEFAULT_PORT;
```

You see! 这里默认配置的是 `localhost` 在spring.redis.host里面的配置根本没有生效。

## 结论

我们需要创建一个自己的`LettuceConnectionFactory` ，正确读取 `spring.redis.host` 里面的配置值。

```java
@Value("${spring.redis.host}")
private String redisHost;

@Value("${spring.redis.port}")
private Integer redisPort;

@Bean
public LettuceConnectionFactory redisConnectionFactory() {
  return new LettuceConnectionFactory(new RedisStandaloneConfiguration(redisHost, redisPort));
}
```