# Identifying API Endpoints

## Introduction

API endpoints can be discovered by browsing applications that use the API.

This is useful even when API documentation is available because documentation may sometimes be **inaccurate or out of date**.

By observing how an application communicates with its API, it is possible to discover endpoints and functionality that may not be documented.

---

## Discovering API Endpoints

### Using Burp Scanner

**Burp Scanner** can be used to crawl an application and identify parts of its attack surface.

After the crawl, interesting areas can be manually investigated using **Burp's browser**.

While browsing the application, look for URL patterns that suggest API endpoints.

For example:

```text
/api/
```

An application may contain URLs such as:

```text
/api/users
/api/products
/api/tasks
```

These patterns can indicate that the application is communicating with an API.

---

## Finding API Endpoints in JavaScript Files

JavaScript files can contain references to API endpoints that have not been directly triggered through normal browser interaction.

For example, an application may contain a JavaScript file that references:

```text
/api/users
/api/admin
/api/tasks
```

even though these endpoints were not accessed while manually using the application's interface.

### Methods for Finding Endpoints in JavaScript

Burp Scanner automatically extracts some endpoints during its crawls.

For more extensive extraction, the **JS Link Finder** BApp can be used.

JavaScript files can also be manually reviewed in Burp to identify references to API endpoints.

---

# Interacting With API Endpoints

Once API endpoints have been identified, they can be investigated using:

* **Burp Repeater**
* **Burp Intruder**

These tools allow you to observe how the API behaves and potentially discover additional attack surface.

For example, you can investigate how an endpoint responds when you change:

* HTTP methods
* Media types

While interacting with endpoints, carefully review:

* Error messages
* HTTP responses
* Other information returned by the server

Error messages and responses may sometimes contain information that helps construct a valid HTTP request.

---

# Identifying Supported HTTP Methods

The **HTTP method** specifies the action that should be performed on a resource.

Some common HTTP methods are:

| Method    | Purpose                                                                 |
| --------- | ----------------------------------------------------------------------- |
| `GET`     | Retrieves data from a resource                                          |
| `PATCH`   | Applies partial changes to a resource                                   |
| `OPTIONS` | Retrieves information about the request methods supported by a resource |

An API endpoint may support multiple HTTP methods.

Therefore, when investigating an endpoint, it is important to test the potential methods it may support.

Different methods can expose different functionality and therefore expand the API's attack surface.

---

## Example

Consider the following endpoint:

```text
/api/tasks
```

It may support different operations:

```http
GET /api/tasks
```

Retrieves a list of tasks.

```http
POST /api/tasks
```

Creates a new task.

```http
DELETE /api/tasks/1
```

Deletes task `1`.

Although these requests are related to the same resource, each method provides different functionality.

---

## Testing HTTP Methods With Burp Intruder

Burp Intruder provides a built-in **HTTP verbs** list that can be used to automatically cycle through a range of HTTP methods.

This can help identify methods supported by an API endpoint.

### Important Consideration

When testing different HTTP methods, use **low-priority objects** where possible.

This helps avoid unintended consequences, such as:

* Altering important items
* Deleting critical data
* Creating excessive records

---

# Identifying Supported Content Types

API endpoints may expect request data in a specific format.

The API can behave differently depending on the **Content-Type** of the request.

Changing the content type may help identify differences in how the API processes the same data.

It may enable you to:

* Trigger errors that disclose useful information.
* Bypass flawed defenses.
* Identify differences in processing logic.

For example, an API may securely process JSON data but have different behavior when processing XML data.

---

## Changing the Content Type

The content type can be changed by modifying the `Content-Type` header.

For example:

```http
Content-Type: application/json
```

can be changed to another supported format, with the request body reformatted accordingly.

The request body must match the format specified by the `Content-Type` header.

---

## Content Type Converter

The **Content type converter** BApp can be used to automatically convert data submitted in requests between formats such as:

```text
JSON
XML
```

This can make it easier to test how an API handles different content types.

---

# Finding Hidden API Endpoints

After identifying some API endpoints, **Burp Intruder** can be used to search for additional hidden endpoints.

For example, suppose reconnaissance identifies:

```http
PUT /api/user/update
```

The `/update` part of the path can be tested with other commonly used API functions.

For example:

```text
/api/user/delete
/api/user/add
/api/user/update
```

The purpose is to discover other endpoints that follow a similar naming structure.

---

## Using API Naming Conventions

When searching for hidden endpoints, use wordlists based on:

* Common API naming conventions
* Common industry terms
* Terms relevant to the specific application

The application's initial reconnaissance can help identify words and functionality that are relevant to the application.

These relevant terms can then be included when searching for additional endpoints.

---

# Overall Process

The process can be summarized as:

```text
Browse the application
        ↓
Identify API patterns
        ↓
Inspect JavaScript files
        ↓
Discover API endpoints
        ↓
Send endpoints to Repeater / Intruder
        ↓
Test HTTP methods
        ↓
Test supported content types
        ↓
Review errors and responses
        ↓
Search for hidden endpoints
```

---

# Key Takeaways

* API endpoints can be discovered by browsing applications that use the API.
* API documentation may be inaccurate or outdated, so observing application behavior is also important.
* URL patterns such as `/api/` can indicate API endpoints.
* JavaScript files may contain references to endpoints that are not directly triggered through the browser.
* Burp Scanner can extract some endpoints during crawling.
* JS Link Finder can perform more extensive extraction from JavaScript files.
* Burp Repeater and Intruder can be used to interact with and investigate API endpoints.
* Testing different HTTP methods can reveal additional functionality.
* The `Content-Type` header can affect how an API processes request data.
* Changing content types can reveal errors, differences in processing logic, or flawed defenses.
* Burp Intruder can be used to search for hidden endpoints using common API naming conventions.
* When testing potentially destructive methods, low-priority objects should be used to avoid unintended consequences.
