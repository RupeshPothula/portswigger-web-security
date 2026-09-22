# API Documentation

## Introduction

API documentation is a reference that explains how an API works and how clients can interact with its endpoints.

It can provide information about:

* Available API endpoints
* HTTP methods
* Parameters
* Request formats
* Response formats
* Authentication requirements
* API versions
* Available resources and functionality

For a security tester, API documentation can be very useful during reconnaissance because it can reveal the application's API attack surface.

---

## What is API Documentation?

API documentation describes the functionality provided by an API.

For example, documentation might describe an endpoint like:

```http
GET /api/users/{id}
```

It may explain:

* What the endpoint does
* Which HTTP method should be used
* Which parameters are required
* What authentication is required
* What response the server returns

A documented API makes it easier for developers and users to understand how to interact with the application.

From a security testing perspective, the same information can help identify endpoints that need further testing.

---

# Why is API Documentation Important for Security Testing?

API documentation can reveal information about an application's functionality that may not be obvious from the normal user interface.

For example, an application might expose:

```text
/api/users
/api/orders
/api/products
/api/admin
```

Some of these endpoints may not have a visible link or button in the application.

By discovering API documentation, a tester can build a better understanding of the available endpoints and functionality.

---

# Common API Documentation Formats

Two commonly encountered API documentation specifications are:

### OpenAPI

OpenAPI is a specification used to describe REST APIs in a structured format.

An OpenAPI document can describe:

* Endpoints
* HTTP methods
* Parameters
* Request bodies
* Responses
* Authentication schemes

It is commonly represented using JSON or YAML.

Example:

```yaml
paths:
  /api/users:
    get:
      summary: Get users
```

---

### SOAP Documentation

SOAP APIs can also have documentation describing their available operations.

SOAP services commonly use **WSDL (Web Services Description Language)** to describe the service.

A WSDL document can provide information about:

* Available operations
* Request structure
* Response structure
* Data types
* Service endpoints

---

# Finding API Documentation

API documentation may be available at predictable locations.

Examples include:

```text
/swagger
/swagger-ui
/openapi.json
/openapi.yaml
/api-docs
```

These locations are only examples. The actual location depends on how the application was developed and configured.

API documentation may also be referenced by the application's source code, JavaScript files, or network requests.

---

# Documentation as a Reconnaissance Source

When API documentation is discovered, I can use it to understand the API before testing it.

For example:

```text
Endpoint:
GET /api/users/{id}

Method:
GET

Parameter:
id

Purpose:
Retrieve information about a user
```

This gives me a structured view of the API.

I can then compare the documented API with the endpoints actually used by the application.

---

# Documented vs Undocumented APIs

An important part of API reconnaissance is understanding that documentation may not represent the complete API.

For example, documentation may show:

```text
GET /api/users
GET /api/products
```

But application traffic might reveal additional endpoints:

```text
GET /api/orders
GET /api/profile
```

Therefore, I should not assume that the documentation contains every available endpoint.

Documentation and observed application traffic can be used together to build a more complete picture of the API.

---

# What to Look for in API Documentation

When reviewing API documentation, I should identify:

### Endpoints

What API endpoints are available?

```text
/api/users
/api/products
/api/orders
```

### HTTP Methods

Which methods are supported?

```text
GET
POST
PUT
PATCH
DELETE
```

### Parameters

What parameters does each endpoint accept?

```text
id
username
productId
orderId
```

### Request Bodies

Does the endpoint accept JSON or another data format?

Example:

```json
{
  "username": "john",
  "email": "john@example.com"
}
```

### Authentication

Does the endpoint require authentication?

Possible mechanisms include:

```text
Session cookies
API keys
Bearer tokens
JWT
```

### Responses

What data does the endpoint return?

Example:

```json
{
  "id": 123,
  "username": "john"
}
```

---

# Using Burp Suite

Burp Suite can help compare API documentation with actual application behavior.

A basic workflow is:

```text
API Documentation
       ↓
Identify endpoints
       ↓
Use the application
       ↓
Capture requests in Burp Suite
       ↓
Compare documented and observed endpoints
       ↓
Understand the API attack surface
```

Burp Suite can also be used to send captured requests to **Repeater**, where requests can be inspected and modified during authorized testing.

---

## Source

This topic was studied as part of the **API Testing** module of PortSwigger Web Security Academy.
