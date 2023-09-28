To create a Spring Boot Cloud Gateway sample app, containerize it using Docker, and configure it to read properties from external variables for rate limiting, routing, and destination route, follow these steps:

1. Create a Spring Boot Cloud Gateway Sample App:

Create a simple Spring Boot Cloud Gateway application with a custom configuration class to handle external properties. You can use the following code as a starting point:

```java
// GatewayProperties.java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "gateway")
public class GatewayProperties {
    private int rateLimit;
    private String defaultRoute;

    // Getter and setter methods for rateLimit and defaultRoute

    // You can add more properties as needed
}
```

2. Configure `application.yml` to use these properties:

```yaml
# application.yml
server:
  port: 8080

spring:
  application:
    name: gateway-service

gateway:
  rateLimit: 100
  defaultRoute: http://localhost:8080
```

3. Create your routing configuration in the application using `RouteLocator`, `RouteDefinition`, or any other approach you prefer.

4. Create a Dockerfile to containerize the application:

```Dockerfile
# Use an official OpenJDK runtime as a parent image
FROM openjdk:11-jre-slim

# Set the working directory to /app
WORKDIR /app

# Copy the built JAR file into the container
COPY target/gateway-service.jar app.jar

# Expose the port that the gateway service will run on
EXPOSE 8080

# Start the Spring Boot application
CMD ["java", "-jar", "app.jar"]
```

5. Build the Docker image using the following command (replace `your-image-name` and `your-image-tag` with your desired image name and tag):

```bash
docker build -t your-image-name:your-image-tag .
```

6. Now, create a script to configure the Cloud Gateway properties at runtime. You can use a shell script for this purpose (e.g., `configure-gateway.sh`):

```bash
#!/bin/bash

# Get the environment variables passed to the container
RATE_LIMIT=${GATEWAY_RATE_LIMIT:-100}
DEFAULT_ROUTE=${GATEWAY_DEFAULT_ROUTE:-http://localhost:8080}

# Replace the properties in the external configuration file
sed -i "s/gateway.rateLimit=$/gateway.rateLimit=$RATE_LIMIT/" /config/application.yml
sed -i "s|gateway.defaultRoute=$|gateway.defaultRoute=$DEFAULT_ROUTE|" /config/application.yml

# Start the Spring Boot application
java -jar /app/app.jar
```

Make sure to grant execute permissions to the script:

```bash
chmod +x configure-gateway.sh
```

7. Update your Dockerfile to copy the `configure-gateway.sh` script and set the entry point:

```Dockerfile
# ...

# Copy the configuration script into the container
COPY configure-gateway.sh /configure-gateway.sh

# Make the script executable
RUN chmod +x /configure-gateway.sh

# Set the entry point
ENTRYPOINT ["/configure-gateway.sh"]
```

8. Build the Docker image again with the updated Dockerfile:

```bash
docker build -t your-image-name:your-image-tag .
```

9. Run the container, passing the desired environment variables:

```bash
docker run -d -p 8080:8080 \
  -e GATEWAY_RATE_LIMIT=500 \
  -e GATEWAY_DEFAULT_ROUTE=http://example.com \
  your-image-name:your-image-tag
```

Now, your Spring Boot Cloud Gateway application will read the configuration from external environment variables at runtime, allowing you to set properties like rate limiting, routing, and destination routes as needed.
