If you want to use a unique value that gets translated or processed before being inserted into the HTML <head> section through IndexHeadInsert, you might need to first set an environment variable with the base64 encoding of UNIQUE_ID in your server configuration and then refer to that variable in the IndexHeadInsert directive.

For instance, in your Apache configuration:

```conf

SetEnv BASE64_UNIQUE_ID %{base64:%{UNIQUE_ID}}


LoadModule autoindex_module modules/mod_autoindex.so
<IfModule mod_autoindex.c>
    IndexOptions +IndexHeadInsert
    IndexHeadInsert "<meta name='CSP_NONCE' content='${BASE64_UNIQUE_ID}'>"
</IfModule>



<IfModule mod_headers.c>
    # Enable the mod_headers module
    Header always set Content-Security-Policy "default-src 'self'; script-src 'nonce-%{UNIQUE_ID}%' 'unsafe-inline'; style-src 'nonce-%{UNIQUE_ID}%' 'unsafe-inline';"

    # Use the UNIQUE_ID environment variable to generate dynamic nonces
    SetEnvIf Request_URI "^" UNIQUE_ID=$RANDOM

    # Insert nonces into inline scripts and styles
    <FilesMatch "\.html$">
        IndexHeadInsert "<script nonce='%{UNIQUE_ID}%'></script>"
        IndexHeadInsert "<style nonce='%{UNIQUE_ID}%'></style>"
    </FilesMatch>
</IfModule>


LoadModule unique_id_module modules/mod_unique_id.so
Header add Content-Security-Policy "default-src 'self' 'localhost' img-src https://\*; 'nonce-%{UNIQUE_ID}e'"

```

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
```
