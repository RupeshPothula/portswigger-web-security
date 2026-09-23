# API Documentation

## Introduction

APIs are usually documented so that developers can understand how to use and integrate with them.

API documentation can provide useful information about the API and can therefore be an important source of information during API reconnaissance.

---

## Types of API Documentation

API documentation can generally be provided in two forms:

### 1. Human-Readable Documentation

Human-readable documentation is designed for developers to understand how to use an API.

It may contain:

* Detailed explanations
* Examples
* Usage scenarios

This makes it easier for developers to understand how the API works and how to integrate it into an application.

---

### 2. Machine-Readable Documentation

Machine-readable documentation is designed to be processed by software.

It can be used for tasks such as:

* Automating API integration
* API validation
* Analyzing API functionality

Machine-readable documentation is commonly written in structured formats such as:

```text
JSON
XML
```

---

## Finding API Documentation

API documentation is often publicly available, especially when an API is intended for use by external developers.

When documentation is publicly available, reviewing it should be an early step in API reconnaissance.

However, API documentation may not always be openly available.

In such cases, it may still be possible to discover documentation by examining applications that use the API.

---

## Discovering API Documentation With Burp Suite

Burp Suite can be used to help discover API documentation.

### Burp Scanner

**Burp Scanner** can crawl an API and help identify endpoints that may contain API documentation.

### Burp's Browser

Applications can also be browsed manually using **Burp's browser**.

While browsing, look for endpoints that may refer to API documentation.

Common examples include:

```text
/api
/swagger/index.html
/openapi.json
```

---

## Investigating Base Paths

When an API resource endpoint is identified, it is useful to investigate the **base paths** leading to that endpoint.

For example, suppose the following resource endpoint is identified:

```text
/api/swagger/v1/users/123
```

Instead of looking only at the complete endpoint, investigate the paths above it:

```text
/api/swagger/v1
/api/swagger
/api
```

This can help identify API documentation located at a higher level in the path structure.

---

## Using Intruder to Discover Documentation

Burp Suite's **Intruder** can also be used with a list of common paths to search for API documentation.

This can help when the documentation is not immediately obvious.

---

## Using Machine-Readable Documentation

Once machine-readable API documentation is discovered, automated tools can be used to analyze it.

### Burp Scanner

Burp Scanner can crawl and audit **OpenAPI documentation** as well as other documentation provided in formats such as:

```text
JSON
YAML
```

### OpenAPI Parser

The **OpenAPI Parser** BApp can be used to parse OpenAPI documentation.

### Specialized API Testing Tools

Specialized tools can also be used to test documented API endpoints.

Examples include:

```text
Postman
SoapUI
```

---

## API Documentation Reconnaissance Flow

```text
Look for API documentation
          ↓
Check publicly available documentation
          ↓
If unavailable, inspect applications using the API
          ↓
Use Burp Scanner / Burp Browser
          ↓
Investigate possible documentation paths
          ↓
Analyze machine-readable documentation
          ↓
Test documented endpoints with suitable tools
```

---

## Key Takeaways

* API documentation helps developers understand how to use and integrate APIs.
* Documentation can be human-readable or machine-readable.
* Machine-readable documentation can use formats such as JSON or XML.
* Public API documentation should be reviewed during reconnaissance.
* API documentation can sometimes be discovered by examining applications that use the API.
* Common documentation paths include `/api`, `/swagger/index.html`, and `/openapi.json`.
* When an API resource is discovered, its base paths should also be investigated.
* Burp Scanner and Burp's browser can help discover API documentation.
* Intruder can be used with common paths to search for documentation.
* Machine-readable documentation can be analyzed using tools such as Burp Scanner and OpenAPI Parser.
