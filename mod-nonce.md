```html
<IfModule mod_headers.c>
  # Set up a variable to hold the nonce value SetEnv nonce %{RANDOM} # Add
  'nonce' attribute to script-src and style-src directives Header always set
  Content-Security-Policy "script-src 'self' 'nonce-%{nonce}'; style-src 'self'
  'nonce-%{nonce}'"
</IfModule>

OR Header always set Content-Security-Policy "script-src 'self' 'nonce-abc123';
style-src 'self' 'nonce-abc123'"
```

and get in bash

```bash
#!/bin/bash

# Generate a nonce
nonce=$(openssl rand -base64 32 | tr -dc 'a-zA-Z0-9' | head -c 16)

# Replace placeholder in httpd.conf with the generated nonce
sed -i "s/{{NONCE_PLACEHOLDER}}/${nonce}/g" /path/to/your/httpd.conf

# Restart Apache (assuming 'httpd' is the Apache service name)
service httpd restart

```
