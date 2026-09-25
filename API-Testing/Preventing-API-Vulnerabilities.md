# Preventing Vulnerabilities in APIs

## Introduction

API security should be considered from the beginning of the API design and development process.

APIs should be designed so that their documentation, HTTP methods, content types, error handling, versions, and user-controlled properties are properly secured.

---

# API Security Practices

## 1. Secure API Documentation

If an API is **not intended to be publicly accessible**, its documentation should also be protected.

Exposing documentation unnecessarily can provide information about the API's attack surface.

Therefore, access to private API documentation should be appropriately secured.

---

## 2. Keep API Documentation Up to Date

API documentation should be kept up to date.

Accurate documentation gives legitimate testers and developers better visibility into the API's available functionality and attack surface.

Outdated documentation can make it difficult to understand which endpoints and functionality are actually supported.

---

## 3. Allowlist HTTP Methods

APIs should use an **allowlist of permitted HTTP methods**.

Only the methods required by an endpoint should be allowed.

For example, if an endpoint only needs to retrieve information, it should not unnecessarily support methods that can modify or delete data.

The basic principle is:

```text
Allow only required HTTP methods
        ↓
Reject unnecessary methods
```

---

## 4. Validate Content Types

APIs should verify that the content type of each request or response is what is expected.

For example, if an endpoint expects JSON:

```http
Content-Type: application/json
```

the application should properly validate that the supplied content type is appropriate.

This helps prevent unexpected processing behavior caused by unsupported or unexpected formats.

---

## 5. Use Generic Error Messages

API error messages should avoid revealing unnecessary technical information.

Detailed error messages can sometimes provide information that may be useful to an attacker.

Instead of exposing internal details, APIs should return appropriate **generic error messages**.

The goal is:

```text
Avoid unnecessary information disclosure
        ↓
Return appropriate generic errors
```

---

# 6. Secure All API Versions

Security protections should be applied to **all versions of an API**, not only the current production version.

For example:

```text
/api/v1/
      ↓
Should be protected

/api/v2/
      ↓
Should be protected

/api/v3/
      ↓
Should be protected
```

An older API version should not be left with weaker security simply because a newer version is currently being used.

---

# Preventing Mass Assignment Vulnerabilities

Mass assignment vulnerabilities can occur when users are able to modify properties of an internal object that they should not control.

Two important approaches can be used to prevent this.

---

## 1. Allowlist Updatable Properties

Create an **allowlist** containing only the properties that users are permitted to update.

For example, if users should only be able to change:

```text
username
email
```

then only those properties should be accepted from the user.

Conceptually:

```text
Allowed:
✓ username
✓ email

Not allowed:
✗ isAdmin
✗ userId
✗ other sensitive properties
```

This limits the fields that can be modified through user input.

---

## 2. Blocklist Sensitive Properties

A **blocklist** can be used to identify properties that users should never be allowed to modify.

For example:

```text
Blocked:
✗ isAdmin
✗ userId
```

This prevents sensitive properties from being changed through API requests.

However, the key protection described here is to **allowlist the properties that users are actually permitted to update**.


---

# Key Takeaways

* API security should be considered from the beginning of development.
* Private API documentation should be properly secured.
* API documentation should remain accurate and up to date.
* Only required HTTP methods should be permitted.
* Expected content types should be validated.
* Error messages should avoid unnecessary information disclosure.
* Security protections should apply to every API version.
* Mass assignment can be prevented by controlling which properties users are allowed to modify.
* Sensitive properties should not be exposed to unrestricted user input.
