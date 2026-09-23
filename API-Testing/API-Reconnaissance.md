# API Reconnaissance

## Introduction

API reconnaissance is the process of gathering as much information as possible about an API before testing it.

The main goal is to understand the API's **attack surface** by identifying its endpoints and determining how those endpoints can be interacted with.

---

## 1. Identifying API Endpoints

An **API endpoint** is a location where an API receives requests for a specific resource on the server.

For example:

```http
GET /api/books HTTP/1.1
Host: example.com
```

The API endpoint in this request is:

```text
/api/books
```

This endpoint could be used to retrieve a list of books from a library.

Another endpoint could be:

```text
/api/books/mystery
```

This could retrieve a list of mystery books.

Therefore, identifying API endpoints is one of the first steps in API reconnaissance.

---

## 2. Understanding How to Interact With Endpoints

After identifying API endpoints, the next step is to determine how to interact with them.

This information helps in constructing valid HTTP requests that can later be used to test the API.

Important information to identify includes:

### Input Data

Determine what input data the API processes.

This includes:

* Compulsory parameters
* Optional parameters

Understanding these parameters helps determine what data can be supplied to an endpoint.

### Supported Requests

Determine what types of requests the API accepts.

This includes:

* Supported HTTP methods
* Supported media formats

For example, an endpoint might accept a particular HTTP method and require data in a specific format.

### Rate Limits

Determine whether the API has any **rate limits**.

Rate limits control how frequently requests can be made to an API.

Understanding the API's rate limits is part of determining how the API can be interacted with.

### Authentication

Identify the **authentication mechanisms** used by the API.

This helps determine what authentication is required when interacting with different endpoints.

---

## API Reconnaissance Process

A basic API reconnaissance process can be summarized as:

```text
Find API endpoints
        ↓
Identify the resources they provide
        ↓
Determine required and optional input
        ↓
Identify supported HTTP methods and media formats
        ↓
Identify rate limits
        ↓
Identify authentication mechanisms
```

---

## Key Takeaways

* API reconnaissance is performed to understand an API's attack surface.
* API endpoints are locations where the API receives requests for specific resources.
* After finding endpoints, it is important to understand how to interact with them.
* Input data can contain compulsory and optional parameters.
* The supported HTTP methods and media formats should be identified.
* Rate limits and authentication mechanisms are also important parts of API reconnaissance.
