# Server-Side Parameter Pollution

## Introduction

Some applications use internal APIs that are not directly accessible from the internet.

A **server-side parameter pollution (SSPP)** vulnerability can occur when an application takes user input and includes it in a server-side request to an internal API without properly encoding the input.

An attacker may then be able to manipulate the server-side request and potentially:

* Override existing parameters.
* Modify application behavior.
* Access unauthorized data.

User input that can be tested for parameter pollution includes:

* Query parameters
* Form fields
* HTTP headers
* URL path parameters

---

## Server-Side Parameter Pollution vs Other Terms

This vulnerability is sometimes referred to as **HTTP parameter pollution**.

However, the term HTTP parameter pollution is also used for a technique involving **Web Application Firewall (WAF) bypasses**.

To avoid confusion, this topic uses the term:

> **Server-side parameter pollution**

Server-side parameter pollution is also different from **server-side prototype pollution**. Despite their similar names, they are separate vulnerability classes.

---

# How Server-Side Parameter Pollution Works

Consider an application that receives user input:

```text
User
  ↓
Web Application
  ↓
Internal API
```

The user cannot directly access the internal API.

However, the web application uses the user's input when constructing a request to the internal API.

For example:

```text
External request
        ↓
Web application
        ↓
Internal API request
```

If the user's input is not properly encoded, the attacker may be able to modify the internal request.

---

# Testing Query String Parameter Pollution

One way to test for SSPP is to place query syntax characters into user input and observe the application's response.

Useful characters include:

```text
#
&
=
```

The response can provide clues about how the server processes the injected characters.

---

# 1. Truncating Query Strings

The URL-encoded `#` character can be used to attempt to **truncate the server-side request**.

Consider an application that allows users to search for other users by username.

The browser sends:

```http
GET /userSearch?name=peter&back=/home
```

The server then makes an internal API request:

```http
GET /users/search?name=peter&publicProfile=true
```

The `publicProfile=true` parameter ensures that only public profiles are returned.

---

## Injecting `#`

The attacker can URL-encode the `#` character:

```http
GET /userSearch?name=peter%23foo&back=/home
```

The application may then construct the internal request as:

```http
GET /users/search?name=peter#foo&publicProfile=true
```

The purpose is to determine whether the `#` causes the remainder of the server-side query to be ignored.

### Why URL-encode `#`?

The `#` character has special meaning in a URL.

If it is sent without encoding, the front-end application may interpret it as a **fragment identifier** and it may not be included in the request to the internal API.

Therefore, use:

```text
%23
```

instead of:

```text
#
```

when testing this behavior through the application's input.

---

## Interpreting the Response

The application's response can provide clues about whether the query was truncated.

For example:

### Response returns the user `peter`

This may indicate that the server-side query was truncated.

### Response returns `Invalid name`

This may indicate that `foo` was treated as part of the username.

That suggests the server-side request may not have been truncated.

---

## Potential Impact

If the server-side request can be truncated, the original:

```text
publicProfile=true
```

parameter may no longer be required.

This could potentially allow access to profiles that were not intended to be publicly accessible.

---

# 2. Injecting Invalid Parameters

The URL-encoded `&` character can be used to attempt to add another parameter to the server-side request.

Original request:

```http
GET /userSearch?name=peter&back=/home
```

The internal request is:

```http
GET /users/search?name=peter&publicProfile=true
```

Now inject:

```http
GET /userSearch?name=peter%26foo=xyz&back=/home
```

The resulting internal request may become:

```http
GET /users/search?name=peter&foo=xyz&publicProfile=true
```

Here, the attacker attempted to inject:

```text
foo=xyz
```

into the internal request.

---

## Interpreting the Response

The response can provide information about how the injected parameter is handled.

If the response is unchanged, this may indicate that:

* The parameter was successfully injected.
* The internal API ignored the parameter.

Further testing is required to understand the behavior.

---

# 3. Injecting Valid Parameters

Once parameter injection is confirmed, you can attempt to inject a **valid parameter** supported by the internal API.

For example, suppose reconnaissance identified an `email` parameter.

The external request could be:

```http
GET /userSearch?name=peter%26email=foo&back=/home
```

This may result in:

```http
GET /users/search?name=peter&email=foo&publicProfile=true
```

The response can then be analyzed to determine how the internal API processes the injected parameter.

The **Finding Hidden Parameters** technique can help identify parameters that may be useful for this type of testing.

---

# 4. Overriding Existing Parameters

Another important test is attempting to inject a second parameter with the **same name** as an existing parameter.

Suppose the original internal request is:

```http
GET /users/search?name=peter&publicProfile=true
```

The attacker can try:

```http
GET /userSearch?name=peter%26name=carlos&back=/home
```

This may produce:

```http
GET /users/search?name=peter&name=carlos&publicProfile=true
```

The internal API now receives two `name` parameters.

---

## How Duplicate Parameters Are Processed

The behavior depends on the technology used by the application.

### PHP

PHP parses the **last parameter only**.

Therefore:

```text
name=peter
name=carlos
```

may result in:

```text
name=carlos
```

### ASP.NET

ASP.NET combines both parameters.

This could result in something similar to:

```text
peter,carlos
```

which may produce an invalid username error.

### Node.js / Express

Node.js / Express parses the **first parameter only**.

Therefore:

```text
name=peter
name=carlos
```

may result in:

```text
name=peter
```

and the response may remain unchanged.

---

## Why This Matters

The way duplicate parameters are processed can determine whether parameter pollution is exploitable.

If the original parameter can be overridden, an attacker may be able to change the application's behavior.

For example:

```http
GET /userSearch?name=peter%26name=administrator
```

could potentially result in an internal request searching for:

```text
name=administrator
```

In an authorized testing environment, this behavior can be investigated to determine its security impact.

---

# Testing Server-Side Parameter Pollution in REST Paths

Not all APIs place parameters in the query string.

A **RESTful API** may place parameter names and values directly in the URL path.

For example:

```text
/api/users/123
```

can be understood as:

```text
/api       → Root API endpoint
/users     → Resource
/123       → User identifier
```

---

## Example

Consider an application that allows users to edit their profile based on their username.

The browser sends:

```http
GET /edit_profile.php?name=peter
```

The application then constructs an internal API request:

```http
GET /api/private/users/peter
```

The user's `name` value is therefore being inserted into the internal API's URL path.

---

# Path Traversal Testing

To test whether the server-side URL path can be manipulated, path traversal sequences can be added to the user input.

For example:

```text
peter/../admin
```

URL-encoded:

```text
peter%2f..%2fadmin
```

The external request becomes:

```http
GET /edit_profile.php?name=peter%2f..%2fadmin
```

The resulting internal request may become:

```http
GET /api/private/users/peter/../admin
```

If the server-side client or back-end API normalizes the path, it may resolve to:

```text
/api/private/users/admin
```

This demonstrates how manipulating a user-controlled path parameter can potentially change which internal resource is requested.

---

# Testing Structured Data Formats

Server-side parameter pollution can also occur when user input is inserted into structured data formats.

Examples include:

```text
JSON
XML
```

The basic idea is:

```text
User input
    ↓
Application
    ↓
Structured data
    ↓
Internal API
```

If the input is not properly validated or encoded, it may alter the structure of the internal request.

---

# JSON Injection Example

Consider an application that allows a user to edit their name.

The browser sends:

```http
POST /myaccount
name=peter
```

The application constructs:

```http
PATCH /users/7312/update
{"name":"peter"}
```

The user input has been inserted into the JSON sent to the internal API.

---

## Injecting an Additional Parameter

An attacker could attempt to supply input such as:

```text
peter","access_level":"administrator
```

The resulting internal request could become:

```json
{
  "name": "peter",
  "access_level": "administrator"
}
```

The attacker has attempted to break out of the original JSON value and introduce a new property.

If the application processes this input without adequate validation or encoding, the injected property may be accepted.

This could potentially result in unauthorized privilege changes.

---

# JSON Input From the Client

The same concept can occur when the user's input is already supplied as JSON.

The browser might send:

```http
POST /myaccount
```

with:

```json
{
  "name": "peter"
}
```

The application then creates:

```http
PATCH /users/7312/update
{"name":"peter"}
```

The attacker can attempt to manipulate the `name` value:

```json
{
  "name": "peter\",\"access_level\":\"administrator"
}
```

If the application decodes the input and inserts it into the server-side JSON without proper encoding, the resulting internal request could become:

```json
{
  "name": "peter",
  "access_level": "administrator"
}
```

Again, this may result in an unauthorized change if the internal API accepts the injected property.

---

# Structured Data Injection in Responses

Structured format injection is not limited to requests.

It can also occur in **responses**.

For example:

```text
User input
    ↓
Stored in database
    ↓
Backend API creates JSON response
    ↓
User receives response
```

If the stored user input is embedded into a JSON response without adequate encoding, it may be possible to manipulate the structure of that response.

The testing approach is similar to structured data injection in requests:

1. Identify user-controlled input.
2. Determine how it is inserted into the structured data.
3. Test whether the input can modify the structure.
4. Analyze the resulting response.

Server-side parameter pollution can occur in other structured data formats as well, not only JSON.

---

# Testing With Automated Tools

Burp Suite provides tools that can help identify potential server-side parameter pollution.

## Burp Scanner

Burp Scanner can automatically detect **suspicious input transformations** during an audit.

This occurs when:

```text
Application receives user input
        ↓
Transforms the input
        ↓
Performs further processing
```

A suspicious input transformation does **not necessarily mean that a vulnerability exists**.

Further manual testing is required to determine whether the behavior is exploitable.

---

## Backslash Powered Scanner

The **Backslash Powered Scanner** BApp can also help identify server-side injection vulnerabilities.

It classifies inputs as:

```text
Boring
Interesting
Vulnerable
```

Inputs classified as **interesting** should be investigated further using the manual testing techniques described in this topic.

---

# Preventing Server-Side Parameter Pollution

Server-side parameter pollution can be prevented by properly controlling and encoding user input before including it in server-side requests.

## 1. Use an Allowlist

Use an **allowlist** to define characters that do not require encoding.

Characters outside the allowed set should be encoded before being included in a server-side request.

---

## 2. Encode User Input

User input should be properly encoded before it is inserted into a server-side request.

This prevents special characters from changing the structure of the request.

---

## 3. Validate Input Format and Structure

Input should also be validated to ensure that it follows the expected:

* Format
* Structure

Unexpected input should not be accepted simply because it can be technically parsed.

---

# Summary

Server-side parameter pollution occurs when user input is inserted into a server-side request to an internal API without adequate encoding.

The main areas to investigate are:

```text
Query strings
     ↓
REST URL paths
     ↓
Structured data
     ↓
JSON / XML
```

Important testing techniques include:

* Injecting `#` to test query-string truncation.
* Injecting `&` to add parameters.
* Injecting valid parameters.
* Attempting to override existing parameters.
* Testing path traversal in REST paths.
* Testing injection into structured data.
* Reviewing responses and errors for useful clues.
* Using Burp Scanner and Backslash Powered Scanner to identify suspicious behavior.

The main defensive principles are:

```text
Validate input
      +
Properly encode input
      +
Use appropriate allowlists
      ↓
Reduce server-side parameter pollution
```
