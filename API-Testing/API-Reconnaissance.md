# API Reconnaissance

## Introduction

API reconnaissance is the process of discovering and understanding an application's APIs before performing security testing.

The main goal is to identify:

* API endpoints
* HTTP methods
* Parameters
* Request and response formats
* Authentication mechanisms
* API versions
* Available functionality

Understanding the API structure helps a security tester determine which parts of the application should be tested further.

---

## What is an API?

An API (Application Programming Interface) allows different software components to communicate with each other.

For example, a web application may use an API to retrieve information about a user's account.

```http
GET /api/users/123
```

The server may respond with:

```json
{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

Here:

* `/api/users/123` is the API endpoint.
* `GET` is the HTTP method.
* `123` is a parameter identifying the user.
* The JSON data is the server's response.

---

# Why is API Reconnaissance Important?

Modern web applications often rely heavily on APIs.

An application may have many endpoints that are not directly visible through the user interface.

For example:

```text
/api/users
/api/users/123
/api/products
/api/orders
/api/admin
/api/v1/users
/api/v2/users
```

Finding these endpoints gives a better understanding of the application's attack surface.

During reconnaissance, I should not only look for endpoints that are visible in the browser. I should also investigate requests made by the application and any available API documentation.

---

# API Attack Surface

The API attack surface consists of the different API endpoints, parameters, methods, and functionality that can potentially be tested.

Important things to identify include:

### 1. Endpoints

Example:

```http
/api/users
/api/products
/api/orders
```

### 2. HTTP Methods

Common methods include:

```text
GET
POST
PUT
PATCH
DELETE
```

Different methods may provide different functionality for the same resource.

### 3. Parameters

Parameters can appear in different locations.

Query parameter:

```http
GET /api/users?id=123
```

Path parameter:

```http
GET /api/users/123
```

Body parameter:

```json
{
  "username": "john"
}
```

### 4. Authentication

I need to determine whether an endpoint requires authentication and what type of authentication is being used.

Examples include:

```text
Session cookies
API keys
Bearer tokens
JWT
```

### 5. API Versions

Applications may expose multiple API versions:

```text
/api/v1/users
/api/v2/users
```

Different versions may have different functionality or security configurations.

---

# API Reconnaissance Techniques

## 1. Inspecting Application Traffic

One of the most useful techniques is observing the requests made by the application.

Using Burp Suite, I can intercept HTTP requests and identify API endpoints.

For example:

```http
GET /api/products/1 HTTP/1.1
Host: example.com
Cookie: session=...
```

From this request, I can identify:

* The API endpoint
* HTTP method
* Host
* Authentication/session information
* Parameters

---

## 2. Using Burp Suite Proxy

Burp Suite's Proxy can be used to intercept requests between the browser and the server.

A basic workflow is:

```text
Browser
   ↓
Burp Suite Proxy
   ↓
Web Application
   ↓
Server
```

While using the application, I can observe requests and identify API endpoints.

---

## 3. Burp Suite HTTP History

Burp Suite's HTTP history is useful for reviewing requests that have already been captured.

I can look for requests containing paths such as:

```text
/api/
/api/v1/
/api/v2/
```

This can reveal APIs that are being used by the application.

---

## 4. Identifying API Endpoints

While reviewing requests, I look for patterns such as:

```text
/api/users
/api/products
/api/orders
/api/account
/api/admin
```

I also pay attention to parameters and HTTP methods associated with each endpoint.

For example:

```http
GET /api/users/123
```

and

```http
DELETE /api/users/123
```

may refer to the same resource but provide completely different functionality.

---


## Source

This topic was studied as part of the **API Testing** module of PortSwigger Web Security Academy.
