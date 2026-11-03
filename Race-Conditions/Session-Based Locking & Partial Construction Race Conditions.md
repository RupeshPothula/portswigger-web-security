# Session-Based Locking & Partial Construction Race Conditions

## 1. Session-Based Locking Mechanisms

### Overview

Some web application frameworks implement session-based locking mechanisms to prevent multiple requests from modifying the same session data simultaneously.

These mechanisms ensure that requests belonging to the same session are processed sequentially rather than concurrently.

For example, PHP's native session handler module uses session locking by default. It generally allows only one request per session to be processed at a time.

### How Session-Based Locking Works

Consider an application where two requests are sent simultaneously using the same session token.

Without locking:
- Both requests may execute concurrently.
- They may access and modify shared session variables.
- Their operations may overlap and produce unexpected results.

With session-based locking:
- The first request acquires a lock on the session.
- The second request must wait until the first request releases the lock.
- Requests are processed sequentially, preventing certain session-related race conditions.

### Impact on Race Condition Testing

Session-based locking can hide vulnerabilities that would otherwise be exploitable.

If all requests appear to execute sequentially, it may indicate that the application is using session-based locking rather than properly preventing the underlying race condition.

**Testing consideration:**

Try sending each request using a different session token.

This allows requests to operate under separate sessions, potentially bypassing the session-level serialization and revealing race conditions in shared application resources.

**Key Takeaway:** Session-based locking can prevent concurrent requests within the same session, but it does not necessarily protect shared database records or application resources from race conditions.

---

## 2. Partial Construction Race Conditions

### Overview

Partial construction race conditions occur when an application creates or initializes an object through multiple separate operations.

Between these operations, the object may temporarily exist in an incomplete or partially initialized state.

An attacker may exploit this temporary state to bypass security checks or access resources before initialization is complete.

### How Partial Construction Race Conditions Work

Consider a user registration process.

The application performs two separate database operations:

1. Creates a new user account in the database.
2. Generates and assigns an API key to that user.

These operations are not executed atomically.

As a result, a small race window exists between the two operations.

During this window:
- The user account already exists.
- The API key has not yet been initialized.
- The database may contain an empty or `NULL` value for the API key.

If the application performs authentication checks against this uninitialized value, an attacker may be able to exploit the incomplete object state.

### 2.1 Exploiting Uninitialized Values

An attacker may attempt to supply an input value that matches the uninitialized database value.

Common examples include:

- Empty strings (`""`)
- `NULL` values
- Empty arrays (`[]`)

If the application's security checks incorrectly accept these values, it may allow unauthorized access.

**Key Concept:** The vulnerability occurs because security-sensitive objects become accessible before all their required properties have been initialized.

### 2.2 Passing Non-String Data Structures Through Parameters

Some web frameworks support passing arrays and other non-string data structures through HTTP parameters using special syntax.

This behavior can be useful when testing partial construction race conditions.

#### PHP Parameter Syntax

PHP supports array parameters through bracket notation.

| HTTP Parameter | Resulting PHP Value |
|---|---|
| `param[]=foo` | `['foo']` |
| `param[]=foo&param[]=bar` | `['foo', 'bar']` |
| `param[]` | `[]` |

These parameter formats allow an attacker to submit an empty array instead of a normal string value.

#### Ruby on Rails Parameter Syntax

Ruby on Rails also supports nested parameters.

For example:

```http
param[key]
```

When a key is provided without a value, Rails can interpret it as a nested object containing `nil`.

The resulting server-side representation is:

```ruby
params = {"param" => {"key" => nil}}
```

This behavior can be important when an application expects a string but receives a `nil` value instead.

### 2.3 Example: API Key Initialization Race

Consider an API that authenticates users using a username and API key.

During normal registration:

1. A user account is created.
2. An API key is generated and assigned.
3. The API uses the username and API key to authenticate requests.

However, during the race window, the account exists while its API key is still uninitialized.

An attacker may attempt to send an API request using an empty array as the API key parameter.

Example:

```http
GET /api/user/info?user=victim&api-key[]= HTTP/2
Host: vulnerable-website.com
```

Here:

- `user=victim` specifies the target account.
- `api-key[]` submits an empty array instead of a normal API key string.
- The application may interpret the supplied value in a way that matches the uninitialized API key.

If the authentication mechanism incorrectly accepts this comparison, the attacker may gain unauthorized access to the victim's account information.

**Important:** Successful exploitation depends on the application's parameter parsing, type comparison, authentication logic, and the timing of the initialization process.

### 2.4 Password-Based Partial Construction Collisions

Similar partial construction race conditions can occur when applications initialize passwords in separate operations.

However, passwords introduce an additional complication because they are normally stored as password hashes rather than plaintext values.

To exploit this situation, the supplied input must produce a hash digest that matches the uninitialized value used by the application.

Therefore, password-based collisions are generally more complicated than API-key-based collisions.

---

## 3. Key Differences

| Feature | Session-Based Locking | Partial Construction |
|---|---|---|
| Main concept | Serializes requests belonging to the same session. | Exploits objects during incomplete initialization. |
| Root cause | Session-level request locking. | Multi-step object creation without atomicity. |
| Main testing focus | Whether requests are processed sequentially. | Whether incomplete objects are temporarily accessible. |
| Common scenario | Multiple requests using one session token. | User registration and API key initialization. |
| Security impact | Can mask underlying race conditions. | May allow authentication or authorization bypass. |

## 4. Key Takeaways

- Session-based locking mechanisms process requests sequentially within the same session.
- PHP's native session handler is an example of session-based locking.
- Testing with different session tokens can help identify race conditions hidden by session locking.
- Partial construction race conditions occur when objects are created or initialized through multiple separate operations.
- A temporary incomplete state may expose uninitialized values such as empty strings or `NULL`.
- PHP and Ruby on Rails support parameter formats that can produce arrays or nested data structures.
- Empty arrays and `nil` values may become useful when testing how authentication mechanisms handle unexpected input types.
- API key initialization and password creation are potential areas to investigate.

## 5. Prevention and Mitigation

Developers should consider the following security practices:

- **Atomic Object Creation:** Create user accounts and initialize security-sensitive properties within a single database transaction.
- **Initialization Validation:** Do not allow an object to be used until all required properties are successfully initialized.
- **Strict Type Validation:** Validate incoming parameters and reject unexpected data types such as arrays when strings are required.
- **Secure Authentication Checks:** Reject empty, `NULL`, or uninitialized API keys and other authentication credentials.
- **Proper Session Handling:** Use appropriate synchronization mechanisms when multiple requests access shared session data.
- **Database Constraints:** Apply `NOT NULL` constraints and other suitable validation rules to security-sensitive database fields.
- **Safe API Design:** Ensure authentication decisions cannot be bypassed through unexpected parameter formats or type coercion.

---

**Related Topic:** [Race Condition Vulnerabilities](https://portswigger.net/web-security/race-conditions)

**Reference:** [PortSwigger Web Security Academy – Race Conditions](https://portswigger.net/web-security/race-conditions)
