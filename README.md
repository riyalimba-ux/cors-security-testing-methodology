# 🌐 CORS Security Testing Methodology

**Cross-Origin Resource Sharing (CORS)** is a browser security mechanism that controls whether a web application can make cross-origin requests and access responses from another origin.

Incorrectly configured CORS policies can expose sensitive resources to unauthorized origins.

This guide explains the fundamentals of CORS, common configuration mistakes, a practical testing methodology, and defensive recommendations.

> ⚠️ Test only applications and APIs where you have explicit authorization.

---

# 🧠 What Is CORS?

Browsers enforce the **Same-Origin Policy (SOP)**, which restricts how a web page from one origin can interact with resources from another origin.

CORS provides a controlled mechanism for allowing specific cross-origin interactions.

An origin consists of:

```text
scheme + host + port
```

For example:

```text
https://example.com
```

and:

```text
https://api.example.com
```

are different origins because their hosts differ.

---

# 🔎 Basic CORS Flow

A browser may send an `Origin` header with a cross-origin request:

```http
GET /api/profile HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
```

The server may respond with:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
```

The browser uses the CORS response headers to determine whether the requesting origin is allowed to access the response.

---

# 🔐 Important CORS Headers

## 1️⃣ Access-Control-Allow-Origin

This header specifies which origins are allowed to access the resource.

Example:

```http
Access-Control-Allow-Origin: https://app.example.com
```

A wildcard configuration looks like:

```http
Access-Control-Allow-Origin: *
```

Whether a wildcard is appropriate depends on the resource and authentication model.

---

## 2️⃣ Access-Control-Allow-Credentials

This controls whether browsers may include credentials in cross-origin requests when the server allows them.

Example:

```http
Access-Control-Allow-Credentials: true
```

Credentials can include mechanisms such as cookies.

A particularly important security consideration is the combination of:

```text
Specific/reflective origin
+
Allow-Credentials: true
```

because an overly permissive policy may expose authenticated responses to an unauthorized origin.

---

## 3️⃣ Access-Control-Allow-Methods

Specifies methods permitted for cross-origin requests.

Example:

```http
Access-Control-Allow-Methods: GET, POST, PUT
```

---

## 4️⃣ Access-Control-Allow-Headers

Specifies request headers that may be used in a cross-origin request.

Example:

```http
Access-Control-Allow-Headers: Content-Type, Authorization
```

---

## 5️⃣ Access-Control-Expose-Headers

Controls which response headers are exposed to browser JavaScript.

Example:

```http
Access-Control-Expose-Headers: X-Request-ID
```

---

# 🧪 CORS Testing Methodology

A basic authorized assessment can follow this process:

```text
Identify API
     ↓
Send Request
     ↓
Add Origin Header
     ↓
Inspect Response
     ↓
Test Origin Validation
     ↓
Check Credentials
     ↓
Review Sensitive Data
     ↓
Assess Actual Impact
```

---

# 1️⃣ Identify Cross-Origin Endpoints

Look for endpoints such as:

```text
/api/profile
/api/account
/api/orders
/api/user
/api/settings
/api/data
```

Pay particular attention to endpoints that return sensitive or user-specific information.

---

# 2️⃣ Send an Origin Header

During an authorized assessment, send a request containing a controlled origin:

```http
GET /api/profile HTTP/1.1
Host: example.com
Origin: https://test.example
```

Then inspect the response.

For example:

```http
Access-Control-Allow-Origin: https://test.example
```

This indicates that the server accepted the supplied origin for CORS purposes.

---

# 3️⃣ Test Origin Validation

Determine whether the application validates origins correctly.

Potential cases include:

```text
Trusted origin
Untrusted origin
Null origin
Subdomain
Similar-looking domain
```

The objective is to understand the application's origin-validation logic.

---

# 4️⃣ Check Credential Handling

Determine whether the response contains:

```http
Access-Control-Allow-Credentials: true
```

If credentials are allowed, carefully assess whether an unauthorized origin can access authenticated data.

The presence of a CORS header alone does **not** automatically mean the application is vulnerable.

---

# 5️⃣ Check Sensitive Responses

Look at what the endpoint actually returns.

For example:

```json
{
  "username": "example",
  "email": "user@example.com",
  "account_type": "standard"
}
```

A permissive CORS configuration on a public endpoint may have little security impact.

A similar configuration on a sensitive authenticated endpoint can have significantly greater consequences.

**Impact matters.**

---

# 🔍 Common CORS Misconfigurations

## ❌ Reflecting Arbitrary Origins

A server may reflect the supplied `Origin` value without properly validating it.

Conceptually:

```text
Request Origin
      ↓
Server accepts it automatically
      ↓
Response reflects Origin
```

This can become dangerous when sensitive authenticated resources are involved.

---

## ❌ Overly Broad Trusted Origins

An application may trust a large range of origins when only a small set is actually required.

For example:

```text
*.example.com
```

may introduce additional considerations if an untrusted or compromised subdomain exists.

---

## ❌ Credentials With Unsafe Origin Policies

Combining credentialed cross-origin access with overly permissive origin validation can create serious security consequences.

The important question is:

> Can an unauthorized website cause the browser to expose authenticated application data?

---

# 🛠️ Testing With Burp Suite

Burp Suite can be useful for inspecting and modifying CORS-related requests.

Basic workflow:

```text
Browser
   ↓
Burp Proxy
   ↓
Capture Request
   ↓
Modify Origin
   ↓
Forward Request
   ↓
Inspect Response
```

Useful Burp features include:

* Proxy
* HTTP history
* Repeater
* Comparer

---

# 🧪 Example Request

```http
GET /api/account HTTP/1.1
Host: example.com
Origin: https://test.example
Cookie: session=AUTHORIZED_TEST_SESSION
```

Example response:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://test.example
Access-Control-Allow-Credentials: true
Content-Type: application/json
```

The next step isn't immediately:

> "This is vulnerable."

Instead, investigate whether the policy allows an unauthorized origin to access sensitive authenticated information.

---

# 🛡️ How Developers Can Prevent CORS Issues

## Use an Explicit Allowlist

Instead of trusting arbitrary origins, define the origins that actually need access.

Conceptually:

```text
Allowed:
https://app.example.com

Not allowed:
https://random.example
```

---

## Avoid Unnecessary Credentialed CORS

Only allow credentials when the application actually requires cross-origin authenticated requests.

---

## Validate Origins Server-Side

Do not blindly reflect the `Origin` header.

The server should compare the supplied origin against a trusted allowlist.

---

## Minimize Exposed Resources

Not every API endpoint needs to support cross-origin access.

Apply CORS policies only where required.

---

# 📋 CORS Testing Checklist

```text
[ ] Identify cross-origin endpoints
[ ] Inspect Origin handling
[ ] Test trusted origin
[ ] Test untrusted origin
[ ] Review wildcard policies
[ ] Check Allow-Credentials
[ ] Review sensitive endpoints
[ ] Check exposed response headers
[ ] Test preflight behavior
[ ] Assess actual data exposure
[ ] Document security impact
[ ] Recommend least-privilege configuration
```

---

# 🧠 Important Distinction

One of the most important lessons when testing CORS is:

```text
CORS Misconfiguration
        ≠
Automatically Vulnerable
```

Always determine:

```text
Misconfiguration
      ↓
What origin is trusted?
      ↓
Are credentials involved?
      ↓
What resource is exposed?
      ↓
Can sensitive data be read?
      ↓
What is the actual impact?
```

This prevents reporting configuration differences that don't create meaningful security impact.

---

# 💡 Key Takeaways

When assessing CORS, don't focus only on whether a header exists.

Ask:

> **Which origins are trusted?**

> **Are credentials allowed?**

> **What data can be accessed?**

> **Can an unauthorized website read the response?**

> **Is the exposed information sensitive?**

A secure CORS configuration follows the principle of **least privilege**: allow only the origins, methods, headers, and credentials that the application genuinely needs.

---

# 📚 Related Topics

* Same-Origin Policy
* HTTP Security Headers
* API Security
* Authentication
* Authorization
* CSRF
* Session Security
* Web Application Security
* Burp Suite
* OWASP

---

# 🚀 Recommended Learning Path

```text
HTTP Fundamentals
       ↓
Same-Origin Policy
       ↓
CORS
       ↓
Cookies & Sessions
       ↓
Authentication
       ↓
Authorization
       ↓
CSRF
       ↓
API Security
```

---

## ⚠️ Responsible Testing

This repository is intended for educational purposes and authorized security testing.

Only test systems you own, intentionally vulnerable environments, CTFs, or systems where you have explicit permission.

⭐ **Learn security. Test responsibly. Build securely.**

#Cybersecurity #CORS #WebSecurity #APISecurity #AppSec #BugBounty #Pentesting #EthicalHacking #BurpSuite #OWASP
