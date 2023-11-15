In Spring Cloud Gateway, enabling OAuth for securing your APIs involves a few steps to set up OAuth authentication using Spring Security and configuring the Gateway routes to authenticate requests. Below are the steps to enable simple OAuth on Spring Cloud API Gateway:

1. **Add Dependencies**:
   Ensure you have the required dependencies in your `pom.xml` file for Spring Security, OAuth2, and Spring Cloud Gateway:

```xml
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Spring Cloud Gateway -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <!-- Spring Security OAuth2 -->
    <dependency>
        <groupId>org.springframework.security.oauth.boot</groupId>
        <artifactId>spring-security-oauth2-autoconfigure</artifactId>
        <version>2.3.8.RELEASE</version> <!-- Check for the latest version -->
    </dependency>
</dependencies>
```

2. **Configure Spring Security**:
   Set up security configurations in your `SecurityConfig` class:

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .oauth2Login();
    }
}
```

This configuration ensures that all requests to your API Gateway routes are authenticated using OAuth2.

3. **Configure OAuth2 Client**:
   In your `application.yml` (or `application.properties`) file, configure your OAuth2 client details:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          your-oauth-client-id:
            client-id: your-client-id
            client-secret: your-client-secret
            authorization-grant-type: authorization_code
            redirect-uri: http://localhost:8080/login/oauth2/code/your-oauth-client-id
            scope: openid,profile,email
        provider:
          your-oauth-provider:
            authorization-uri: https://your-oauth-provider.com/oauth/authorize
            token-uri: https://your-oauth-provider.com/oauth/token
            user-info-uri: https://your-oauth-provider.com/userinfo
            user-name-attribute: name
```

4. **Configure Spring Cloud Gateway**:
   Configure your API routes in `application.yml` to enforce OAuth authentication:

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: your-route-id
          uri: http://your-service-uri
          predicates:
            - Path=/your-service/**
          filters:
            - RewritePath=/your-service/(?<segment>.*), /$\{segment}
            - OAuth2
```

Replace `your-oauth-client-id`, `your-client-id`, `your-client-secret`, `your-oauth-provider`, `your-service-uri`, and other placeholders with your actual OAuth client details, provider information, and service endpoints.

5. **Run and Test**:
   Run your Spring Boot application and test accessing your API routes via the Gateway. Requests should be redirected to the OAuth provider's authentication page for login before accessing the protected routes.

Remember to handle tokens, user authentication, and authorization based on your application's requirements and business logic. Adjust configurations as needed for more advanced scenarios or specific OAuth providers.
