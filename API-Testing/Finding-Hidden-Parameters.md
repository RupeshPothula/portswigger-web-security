# Finding Hidden Parameters

## Introduction

During API reconnaissance, an API may support **undocumented parameters** that are not visible in the normal API documentation or requests.

These hidden parameters can sometimes change how the application behaves.

Finding and testing these parameters is therefore an important part of API reconnaissance.

---

# Finding Hidden Parameters

Burp Suite provides several tools that can help identify hidden parameters.

## 1. Burp Intruder

**Burp Intruder** can be used to automatically search for hidden parameters.

It can use a wordlist containing common parameter names and:

* Replace existing parameters
* Add new parameters

For example, if an API request contains:

```json
{
  "username": "wiener"
}
```

you could use a wordlist containing possible parameter names such as:

```text
email
id
role
isAdmin
status
```

The parameter names can be tested to determine whether the API recognizes any of them.

It is also useful to include parameter names that are relevant to the specific application based on information discovered during initial reconnaissance.

---

## 2. Param Miner

**Param Miner** is a Burp BApp that can automatically guess parameter names.

It can guess up to **65,536 parameter names per request**.

Param Miner can also automatically generate parameter names that are relevant to the application using information gathered from the defined scope.

---

## 3. Content Discovery

Burp's **Content discovery** tool can be used to find content that isn't linked from the visible application.

This can include hidden parameters as well as other content that is not directly accessible through normal navigation.

---

# Mass Assignment Vulnerabilities

One important reason hidden parameters may exist is **mass assignment**.

Mass assignment is also known as **auto-binding**.

It occurs when a software framework automatically binds request parameters to fields on an internal object.

Because of this automatic binding, the application may accept parameters that the developer did not intend users to control.

For example, an application may intend users to update only:

```text
username
email
```

But the internal user object may also contain:

```text
id
isAdmin
```

If the framework automatically binds request parameters to object fields, these additional fields may become accessible through the API.

---

# Identifying Hidden Parameters in Mass Assignment

One way to identify potential hidden parameters is to compare the fields accepted by an API request with the fields returned by the API.

Consider the following request:

```http
PATCH /api/users/
```

The request allows the user to update their username and email:

```json
{
  "username": "wiener",
  "email": "wiener@example.com"
}
```

Now consider a separate request:

```http
GET /api/users/123
```

which returns:

```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "isAdmin": "false"
}
```

The response contains fields that were not included in the original `PATCH` request.

In particular:

```text
id
isAdmin
```

These fields may potentially be bound to the internal user object.

Therefore, they are worth investigating as possible hidden parameters.

---

# Testing for Mass Assignment

Once a potentially hidden parameter has been identified, it can be added to the request to determine how the application handles it.

For example:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false
}
```

The response and application behavior can then be compared with the original request.

---

## Testing With an Invalid Value

An invalid value can also be supplied:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": "foo"
}
```

The behavior can then be compared with the request containing the valid value.

If the application behaves differently when an invalid value is supplied, this may provide information about how the parameter is processed.

If the invalid value affects the query logic while the valid value does not produce an obvious difference, this may indicate that the parameter is being processed by the application.

This can be a clue that the parameter may be successfully updated.

---

# Testing the Parameter's Effect

If the parameter appears to be accepted, it can be tested with another value in an authorized lab or testing environment.

For example:

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": true
}
```

The purpose is to determine whether the application actually binds the parameter to the internal object.

If the application accepts the parameter without adequate validation or sanitization, the user could potentially be granted privileges that they should not have.

For example:

```text
isAdmin = false
      ↓
isAdmin = true
      ↓
Unexpected administrative access
```

---

# Verifying the Behavior

Simply receiving a successful HTTP response does not necessarily prove that the parameter was successfully modified.

The application's actual behavior should be checked.

In the example above, the application can be browsed as the affected user to determine whether administrative functionality has become accessible.

The important question is:

> **Did changing the hidden parameter actually change the user's privileges or application behavior?**

---

# Mass Assignment Testing Process

The overall process can be summarized as:

```text
Identify API request
        ↓
Examine API response objects
        ↓
Look for fields not present in the request
        ↓
Identify potential hidden parameters
        ↓
Add the parameter to the request
        ↓
Test valid and invalid values
        ↓
Compare application behavior
        ↓
Verify whether the parameter actually changed the object
```

---

# Key Takeaways

* APIs may support undocumented parameters.
* Burp Intruder can be used with parameter wordlists to discover hidden parameters.
* Param Miner can automatically guess large numbers of parameter names.
* Content discovery can help identify hidden content, including parameters.
* Mass assignment occurs when frameworks automatically bind request parameters to fields on internal objects.
* Mass assignment can expose parameters that developers did not intend users to control.
* Comparing API requests with API response objects can help identify potential hidden parameters.
* Parameters such as `isAdmin` should be investigated carefully in an authorized testing environment.
* Testing both valid and invalid parameter values can help understand how the application processes them.
* The actual application behavior should be verified rather than relying only on the HTTP response.
