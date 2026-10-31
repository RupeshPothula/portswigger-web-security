# Single-Endpoint Race Conditions

## 1. Overview

Single-endpoint race conditions occur when multiple parallel requests are sent to the same application endpoint with different input values.

If the application processes these requests concurrently without properly synchronizing shared data, the operations may interfere with each other and produce unexpected results.

These vulnerabilities are particularly dangerous when multiple requests modify the same session variables, account information, or application state.

## 2. How Single-Endpoint Race Conditions Work

Consider a password reset mechanism that stores the user's ID and password reset token in the user's session.

An attacker may send two parallel password reset requests from the same session, using two different usernames:

- **Request 1:** Initiates a password reset for the attacker's account.
- **Request 2:** Initiates a password reset for the victim's account.

Both requests attempt to modify the same session variables:

- `session['reset-user']`
- `session['reset-token']`

Because the requests execute concurrently, their individual operations may overlap.

### Example: Password Reset Collision

The application may perform the following operations:

1. Request 1 sets the session's reset user to the attacker's username.
2. Request 2 changes the reset user to the victim's username.
3. Request 1 generates a valid reset token.
4. Request 1 sends the reset token to the attacker.

Depending on the execution order, the final session state could become:

```python
session['reset-user'] = victim
session['reset-token'] = 1234
```

In this situation:

- The session contains the victim's user ID.
- The valid reset token is delivered to the attacker.

This collision may allow the attacker to use the reset token associated with the victim's account, potentially resulting in account takeover.

**Key Concept:** The vulnerability occurs because separate concurrent operations modify shared session data without proper synchronization.

## 3. Importance of Execution Order

Single-endpoint race conditions depend heavily on the order in which individual operations are executed.

For the password reset collision to succeed, the application must process the operations in a specific sequence that produces the required combination of session values.

Important considerations:

- Requests may execute in unpredictable orders.
- Individual operations may overlap.
- The desired session state may not occur on every attempt.
- Multiple attempts may be required to trigger the vulnerability.

Therefore, understanding the application's internal processing sequence is important when investigating these race conditions.

## 4. Common Targets for Single-Endpoint Race Conditions

Email-based functionality is particularly interesting when testing for single-endpoint race conditions.

Examples include:

- Password reset emails.
- Email address confirmation.
- Account registration and verification.
- Email change functionality.
- Account recovery mechanisms.

### Why Email Operations Are Vulnerable

Email operations are often handled asynchronously using background threads or tasks.

In some applications, the server sends the HTTP response before the email operation has completely finished.

This creates an opportunity for another request to execute while the first request is still processing its background operations.

As a result, multiple requests may interfere with shared application state, increasing the possibility of race condition vulnerabilities.

## 5. Key Takeaways

- Single-endpoint race conditions involve sending concurrent requests to the same endpoint with different input values.
- They commonly occur when multiple requests modify shared session variables or application state.
- Password reset mechanisms can be vulnerable when user identifiers and reset tokens are stored separately in a shared session.
- An incorrect execution order can cause a token to be associated with the wrong user.
- Exploitation depends on precise timing and operation order, so multiple attempts may be necessary.
- Email-based operations are interesting testing targets because they frequently use asynchronous background processing.
- Proper synchronization and secure session management are essential to prevent these vulnerabilities.

## 6. Prevention and Mitigation

Developers can prevent single-endpoint race conditions through the following measures:

- **Atomic Operations:** Ensure related operations, such as assigning a user ID and generating a reset token, execute atomically.
- **Proper Session Management:** Avoid storing sensitive, independently updated authentication data in shared session variables without synchronization.
- **Request Synchronization:** Prevent concurrent requests from modifying the same sensitive application state inconsistently.
- **Secure Token Binding:** Bind password reset tokens directly to the intended user account instead of relying on mutable session variables.
- **Asynchronous Task Safety:** Ensure background email operations use immutable, request-specific data.
- **Server-Side Validation:** Verify that the reset token belongs to the intended account before allowing a password change.

---

**Related Topic:** [Race Condition Vulnerabilities](https://portswigger.net/web-security/race-conditions)

**Reference:** [PortSwigger Web Security Academy – Race Conditions](https://portswigger.net/web-security/race-conditions)
