LoadModule unique_id_module modules/mod_unique_id.so
Header add Content-Security-Policy "default-src 'self' 'localhost' img-src https://\*; 'nonce-%{UNIQUE_ID}e'"

````

Then curl it curl -sD - localhost -o /dev/null

https://github.com/wyday/mod_cspnonce/releases/tag/1.4

```yaml
spring:
  application:
    name: your-application-name
  cloud:
    gateway:
      routes:
        - id: example_route
          uri: https://example.com
          predicates:
            - Path=/example/**
      # Other Spring Cloud Gateway configurations here

logging:
  file:
    name: mylog.log
    max-size: 10MB
    max-history: 5
  level:
    org.springframework.cloud.gateway: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
````
