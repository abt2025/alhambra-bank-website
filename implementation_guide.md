# Alhambra Bank & Trust - Security Vulnerability Fixes and Implementation Guide

## 1. Introduction

This document provides a detailed guide to fixing the security vulnerabilities identified in the Alhambra Bank & Trust website. It includes code examples and step-by-step instructions to implement the necessary security enhancements.

## 2. HTML Code Fixes

### 2.1. Subresource Integrity (SRI)

To prevent the loading of malicious scripts from a compromised CDN, add the `integrity` and `crossorigin` attributes to the Tailwind CSS script tag.

**Original Code:**
```html
<script src="https://cdn.tailwindcss.com"></script>
```

**Fixed Code:**
```html
<script src="https://cdn.tailwindcss.com" integrity="sha384-igm5BeiBt36UU4gqwWS7imYmelpTsZlQ45FZf+XBn9MuJbn4nQr7yx1yFydocC/K" crossorigin="anonymous"></script>
```

### 2.2. Sensitive Data Exposure

Remove hardcoded email addresses from the source code to prevent email harvesting.

**Original Code:**
```javascript
"email": "abt@abt.ky"
```

**Fixed Code:**
```javascript
"email": "[email protected]"
```

### 2.3. Cross-Site Scripting (XSS)

While no direct `innerHTML` or `eval` were found in the final cleaned code, it is crucial to ensure that any dynamic content is handled safely. Always use `textContent` instead of `innerHTML` when inserting text into the DOM.

**Example:**
```javascript
// Unsafe
document.getElementById("example").innerHTML = userInput;

// Safe
document.getElementById("example").textContent = userInput;
```

## 3. Server-Side Security

### 3.1. HTTP to HTTPS Redirection

All HTTP traffic should be redirected to HTTPS to ensure encrypted communication. This can be configured on your web server.

### 3.2. Security Headers

Implement the following security headers to protect against common web vulnerabilities.

#### Nginx Configuration (`nginx.conf`)

```nginx
server {
    listen 80;
    server_name your_domain.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name your_domain.com;

    # SSL/TLS configuration
    ssl_certificate /path/to/your/certificate.pem;
    ssl_certificate_key /path/to/your/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # Security Headers
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' https://cdn.tailwindcss.com; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self';";
    add_header X-Frame-Options "DENY";
    add_header X-Content-Type-Options "nosniff";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "no-referrer";
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()";

    # Hide server version
    server_tokens off;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### Apache Configuration (`.htaccess`)

```apache
<IfModule mod_headers.c>
    # Security Headers
    Header set Content-Security-Policy "default-src 'self'; script-src 'self' https://cdn.tailwindcss.com; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self';"
    Header set X-Frame-Options "DENY"
    Header set X-Content-Type-Options "nosniff"
    Header set Strict-Transport-Security "max-age=31536000; includeSubDomains" env=HTTPS
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "no-referrer"
    Header set Permissions-Policy "geolocation=(), microphone=(), camera=()"
</IfModule>

<IfModule mod_rewrite.c>
    # Redirect HTTP to HTTPS
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</IfModule>

# Hide server signature
ServerSignature Off
ServerTokens Prod
```

### 3.3. Cross-Site Request Forgery (CSRF)

For any forms that perform state-changing actions, implement CSRF tokens. This involves generating a unique token on the server, embedding it in the form, and verifying it on submission.

**Example (PHP):**

```php
// Generate and store token in session
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$csrf_token = $_SESSION['csrf_token'];

// Add token to form
<input type="hidden" name="csrf_token" value="<?php echo $csrf_token; ?>">

// Verify token on submission
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!isset($_POST['csrf_token']) || !hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
        // Handle CSRF error
        die('CSRF token validation failed.');
    }
}
```

## 4. Conclusion

By implementing these fixes, you will significantly improve the security posture of the Alhambra Bank & Trust website. It is crucial to follow these recommendations to protect the bank and its clients from potential cyber threats. Regular security audits and updates are also recommended to maintain a high level of security.

## 5. References

- [1] [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [2] [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [3] [Subresource Integrity (SRI)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)
- [4] [Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

