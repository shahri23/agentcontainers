To implement in-memory rate limiting in Spring Cloud Gateway, you can use a custom filter along with an in-memory data structure to track and control request rates. Here's an example of configuring rate limiting in the application.yaml file without using Redis:

```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - name: RequestRateLimiter
          args:
            key-resolver: "#{@apiKeyResolver}" # Define your own key resolver bean if needed
            rate-limiter: in-memory
            default-replenish-rate: 10
            default-burst-capacity: 20
      routes:
        - id: example_route
          uri: http://example.com
          predicates:
            - Path=/example/**
```
