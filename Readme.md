curl -X POST --data-binary @file_to_upload.txt http://your_apache_server/endpoint

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

\*\*1
You can parameterize the values in the `application.properties` file by referencing environment variables set in your Bash environment. Here's the same configuration as before, but with environment variable references:

```properties
# Application Port
server.port=${SERVER_PORT}

# Spring Cloud Gateway Routes
spring.cloud.gateway.routes[0].id=example-route
spring.cloud.gateway.routes[0].uri=${EXAMPLE_URI}
spring.cloud.gateway.routes[0].predicates[0]=Path=/example/**
spring.cloud.gateway.routes[0].predicates[1]=Host=${EXAMPLE_HOST}
spring.cloud.gateway.routes[0].filters[0]=RewritePath=/example/(?<segment>.*), /$\{segment}
spring.cloud.gateway.routes[0].filters[1]=AddResponseHeader=X-Response-Header, Gateway Header
spring.cloud.gateway.routes[0].filters[2]=AddRequestHeader=X-Request-Header, Gateway Request Header

# Global Filter Configuration
spring.cloud.gateway.globalfilters[0]=${GLOBAL_FILTER}

# SSL/TLS Configuration
server.ssl.key-store=${SSL_KEYSTORE}
server.ssl.key-store-password=${SSL_KEYSTORE_PASSWORD}
server.ssl.keyAlias=${SSL_KEY_ALIAS}
server.ssl.key-password=${SSL_KEY_PASSWORD}
server.ssl.enabled=${SSL_ENABLED}

# HTTPS Headers
server.ssl.enabled=true
security.require-ssl=true
security.headers.hsts=Max-Age=31536000 ; includeSubDomains
security.headers.hsts.enabled=true
security.headers.hsts.includeSubDomains=true
security.headers.frame=true

# X-Forwarded Headers
server.use-forward-headers=true
spring.cloud.gateway.forwarded.enabled=true

# CORS Configuration
spring.webflux.hiddenmethod.filter.enabled=false
spring.webflux.gzip.enabled=true
spring.webflux.gzip.minResponseSize=10240
spring.webflux.gzip.mimeTypes=text/plain, text/html, text/xml, text/css, application/javascript
spring.cloud.gateway.default-filters[0]=TokenRelay
spring.cloud.gateway.routes[1].id=cors-route
spring.cloud.gateway.routes[1].uri=${CORS_URI}
spring.cloud.gateway.routes[1].predicates[0]=Path=/cors/**
spring.cloud.gateway.routes[1].filters[0]=Cors=/
```

In this updated configuration:

- `${SERVER_PORT}`, `${EXAMPLE_URI}`, `${EXAMPLE_HOST}`, `${GLOBAL_FILTER}`, `${SSL_KEYSTORE}`, `${SSL_KEYSTORE_PASSWORD}`, `${SSL_KEY_ALIAS}`, `${SSL_KEY_PASSWORD}`, and `${SSL_ENABLED}` are placeholders that reference environment variables.
- You should set these environment variables in your Bash environment before running your Spring Boot application.

For example, you can set the `EXAMPLE_URI` environment variable like this:

```bash
export EXAMPLE_URI=https://example.com
```

And similarly, set the other environment variables accordingly.

When you run your Spring Boot application, it will use the values of the referenced environment variables for configuration. This allows you to externalize and parameterize your configuration using environment variables.

\*\*2- Certainly! Here's an updated Bash script that generates the `application.properties` file with sections for routes, predicates, filters, TLS/SSL settings, HTTPS headers, X-Forwarded headers, and CORS configuration based on user-provided environment variables:

```bash
#!/bin/bash

# Function to append a section to the properties file
append_section() {
  echo "" >> application.properties
  echo "$1" >> application.properties
}

# Generate the minimal configuration
cat <<EOL > application.properties
# Application Port
server.port=8080

# Spring Cloud Gateway Routes
spring.cloud.gateway.routes[0].id=default-route
spring.cloud.gateway.routes[0].uri=https://default-backend.com
spring.cloud.gateway.routes[0].predicates[0]=Path=/**
EOL

# Check if ENABLE_CORS is true and append CORS section
if [[ "$ENABLE_CORS" == "true" ]]; then
  append_section "# CORS Configuration"
  append_section "spring.cloud.gateway.routes[1].id=cors-route"
  append_section "spring.cloud.gateway.routes[1].uri=\${CORS_URI}"
  append_section "spring.cloud.gateway.routes[1].predicates[0]=Path=/cors/**"
  append_section "spring.cloud.gateway.routes[1].filters[0]=Cors=/"
fi

# Check if ENABLE_TLS is true and append TLS section
if [[ "$ENABLE_TLS" == "true" ]]; then
  append_section "# TLS/SSL Configuration"
  append_section "server.ssl.key-store=\${SSL_KEYSTORE}"
  append_section "server.ssl.key-store-password=\${SSL_KEYSTORE_PASSWORD}"
  append_section "server.ssl.keyAlias=\${SSL_KEY_ALIAS}"
  append_section "server.ssl.key-password=\${SSL_KEY_PASSWORD}"
  append_section "server.ssl.enabled=true"

  append_section "# HTTPS Headers"
  append_section "security.require-ssl=true"
  append_section "security.headers.hsts=Max-Age=31536000 ; includeSubDomains"
  append_section "security.headers.hsts.enabled=true"
  append_section "security.headers.hsts.includeSubDomains=true"
  append_section "security.headers.frame=true"

  append_section "# X-Forwarded Headers"
  append_section "server.use-forward-headers=true"
  append_section "spring.cloud.gateway.forwarded.enabled=true"
fi

# Add custom routes, filters, and predicates here based on your requirements

echo "Generated application.properties file with optional sections."

# You can then trigger the redeployment of the Gateway pod here.
# For example, if you're using `oc` to manage OpenShift resources:
# oc rollout latest dc/gateway-deployment-config
```

This script generates the `application.properties` file with sections for CORS configuration, TLS/SSL configuration, HTTPS headers, and X-Forwarded headers based on the `ENABLE_CORS` and `ENABLE_TLS` environment variables. You can further customize the script to add custom routes, filters, and predicates as needed.

If you don't know the specific pod name beforehand and want to restart a pod for an application managed by a deployment, you can use the following approach:

1. Identify the deployment's name.
2. Use OpenShift's Rolling Restart Strategy for the deployment.

Here's an updated script to achieve this:

```bash
#!/bin/bash

# Define OpenShift API server URL and credentials
OPENSHIFT_URL="https://your-openshift-cluster"
OPENSHIFT_TOKEN="your-openshift-token"
NAMESPACE="your-namespace"
CONFIG_MAP_NAME="your-configmap-name"
DEPLOYMENT_NAME="your-deployment-name"

# Define the YAML configuration
YAML_FILE="spring_cloud_gateway_properties.yaml"

# Create the YAML content
cat > "$YAML_FILE" <<EOL
spring:
  cloud:
    gateway:
      routes:
        - id: route1
          uri: http://example.com
          predicates:
            - Path=/foo/**
          filters:
            - name: RequestRateLimiter
              args:
                key-resolver: '#{@keyResolver}'
                redis-rate-limiter.replenishRate: 1
                redis-rate-limiter.burstCapacity: 2
                redis-rate-limiter.requestedTokens: 1
            - name: RewritePath
              args:
                regexp: /foo/(?<segment>.*)
                replacement: /\$\{segment}
      default-filters:
        - name: RequestRateLimiter
          args:
            key-resolver: '#{@keyResolver}'
            redis-rate-limiter.replenishRate: 1
            redis-rate-limiter.burstCapacity: 2
            redis-rate-limiter.requestedTokens: 1
EOL

echo "Generated $YAML_FILE with Spring Cloud Gateway configuration"

# Check if the YAML file exists
if [ -f "$YAML_FILE" ]; then
  echo "Creating ConfigMap $CONFIG_MAP_NAME in namespace $NAMESPACE"

  # Encode the YAML file as base64
  ENCODED_YAML=$(cat "$YAML_FILE" | base64)

  # Define the ConfigMap JSON
  JSON_DATA='{
    "kind": "ConfigMap",
    "apiVersion": "v1",
    "metadata": {
      "name": "'"$CONFIG_MAP_NAME"'"
    },
    "data": {
      "spring_cloud_gateway_properties.yaml": "'"$ENCODED_YAML"'"
    }
  }'

  # Create or update the ConfigMap using the OpenShift API
  curl -X PUT "$OPENSHIFT_URL/api/v1/namespaces/$NAMESPACE/configmaps/$CONFIG_MAP_NAME" \
    -H "Authorization: Bearer $OPENSHIFT_TOKEN" \
    -H "Content-Type: application/json" \
    -d "$JSON_DATA"

  # Identify the deployment name
  DEPLOYMENT_NAME=$(oc get deployments -n "$NAMESPACE" --no-headers | awk '{print $1}')

  if [ -n "$DEPLOYMENT_NAME" ]; then
    # Perform a rolling restart of the identified deployment
    echo "Performing a rolling restart of deployment $DEPLOYMENT_NAME in namespace $NAMESPACE"
    curl -X POST "$OPENSHIFT_URL/apis/apps/v1/namespaces/$NAMESPACE/deployments/$DEPLOYMENT_NAME/rollback" \
      -H "Authorization: Bearer $OPENSHIFT_TOKEN" \
      -H "Content-Type: application/json" \
      --data '{"kind":"DeploymentRollback","name":"'"$DEPLOYMENT_NAME"'"}'
  else
    echo "Error: Deployment not found in namespace $NAMESPACE."
  fi
else
  echo "Error: YAML file not found."
fi
```

In this script, we identify the deployment name in the specified namespace, and then we perform a rolling restart of the deployment to ensure that the changes from the updated ConfigMap take effect. This approach is more flexible as it doesn't rely on a specific pod name, and it works with deployments. Make sure to replace the placeholders with your actual OpenShift cluster details and configuration.
