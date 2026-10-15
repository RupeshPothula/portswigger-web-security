# Race Condition Vulnerabilities

## Introduction

A **race condition** is a vulnerability that occurs when a web application processes multiple requests concurrently without properly controlling access to shared data or resources. When requests overlap unexpectedly, the application may behave in an unintended way, allowing attackers to exploit flaws in its business logic.

Race conditions are closely related to **business logic vulnerabilities**, as they exploit the way an application handles operations, state changes, and request processing rather than necessarily exploiting a traditional coding error.

### Key Concepts

- **Concurrent Requests:** Multiple requests being processed at the same time.
- **Race Window:** The short period during which overlapping requests can cause an unintended result. It may last only a few milliseconds.
- **Collision:** An unintended interaction between concurrent requests that affects the application's behavior.
- **Temporary Sub-state:** An intermediate application state that exists while an operation is being processed, before the operation is fully completed.

### How Race Conditions Occur

A race condition can occur when an application checks a condition, performs an operation, and updates its state without ensuring that these steps are handled safely as a single operation.

For example, an application may check whether a discount code has already been used, apply the discount, and then update the database. If two requests perform the initial check before either request updates the database, both may apply the discount.

### Potential Impact

Depending on the application's functionality, race conditions can lead to:

- Redeeming single-use gift cards or discount codes multiple times.
- Performing unauthorized or repeated transactions.
- Withdrawing or transferring more money than the available balance.
- Bypassing rate limits or reusing CAPTCHA solutions.
- Manipulating ratings or other restricted actions.

### Detection and Prevention

**Detection:**
- Identify endpoints that enforce single-use restrictions, rate limits, or other business logic constraints.
- Observe how the application behaves when requests overlap.
- Use tools such as Burp Suite Repeater and Turbo Intruder to investigate race conditions in authorized testing environments.

**Prevention:**
- Use database transactions and appropriate locking mechanisms.
- Enforce critical business rules atomically on the server.
- Apply unique constraints where appropriate.
- Avoid separating a security-sensitive check from the operation it protects without concurrency safeguards.

## Further Reading

- [PortSwigger Web Security Academy — Race Conditions](https://portswigger.net/web-security/race-conditions)

