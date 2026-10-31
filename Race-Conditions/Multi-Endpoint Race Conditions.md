# Multi-Endpoint Race Conditions

## 1. Overview

Multi-endpoint race conditions occur when an application processes multiple requests to different endpoints simultaneously, allowing an attacker to manipulate the timing and order of operations.

These vulnerabilities arise when different endpoints interact with the same application state without proper synchronization or atomic operations.

A classic example involves an online shopping application where:

1. A user adds an item to their shopping cart.
2. The user proceeds to payment and completes the payment process.
3. Before the order is confirmed, additional items are added to the cart.
4. The order confirmation process uses the modified cart contents.

This can allow an attacker to receive additional items without paying for them.

## 2. How Multi-Endpoint Race Conditions Work

A variation of the shopping cart vulnerability occurs when payment validation and order confirmation are handled during a single request.

The application may follow this sequence:

1. The server validates whether the payment is successful.
2. The application enters a race window before confirming the order.
3. Another request modifies the shopping cart during this window.
4. The server confirms the order using the updated cart contents.

If the application does not properly synchronize these operations, an attacker may exploit the timing gap to manipulate the final order.

**Key Concept:** The vulnerability exists because payment validation and order confirmation are not performed as one atomic operation.

## 3. Aligning Multi-Endpoint Race Windows

When testing multi-endpoint race conditions, aligning the execution timing of requests can be challenging.

Even when multiple requests are sent simultaneously using the **Single-Packet Attack** technique, their race windows may not overlap as expected.

This happens primarily because of two factors:

### A. Delays Introduced by Network Architecture

Network infrastructure can introduce delays during request processing.

Examples include:

- Delays when the front-end server establishes a new connection to the back-end.
- Differences in connection handling.
- Protocol-related processing delays.
- Network latency and jitter.

These delays can affect the timing and synchronization of concurrent requests.

### B. Delays Introduced by Endpoint-Specific Processing

Different endpoints may require different amounts of processing time.

For example:

- One endpoint may perform a simple database lookup.
- Another may trigger multiple database operations.
- An endpoint may perform payment validation or external service communication.

As a result, requests sent simultaneously may reach their critical execution points at different times.

## 4. Connection Warming

Connection warming is a technique used to reduce delays caused by back-end connection establishment.

The idea is to send one or more harmless requests before executing the main attack requests.

### How It Works

1. Send an inconsequential request, such as a `GET` request to the application's homepage.
2. Allow the application to establish the necessary back-end connection.
3. Send the actual requests using the warmed connection.
4. Observe whether the processing times become more consistent.

In **Burp Suite Repeater**, this can be performed by:

- Adding a harmless `GET` request to the beginning of a tab group.
- Selecting **Send group in sequence (single connection)**.
- Observing the response times of subsequent requests.

If the first request takes longer but the remaining requests execute within a shorter time window, the initial delay may be caused by connection establishment rather than endpoint-specific processing.

### Important Consideration

Connection warming helps distinguish connection-establishment delays from actual processing delays.

If inconsistent response times remain on a single endpoint, even when using the Single-Packet Attack technique, back-end delays may still be interfering with the attack.

In such situations, **Turbo Intruder** can be used to send connection-warming requests before executing the main attack requests.

## 5. Abusing Rate or Resource Limits

If connection warming does not resolve timing inconsistencies, another technique involves exploiting server-side rate or resource limits to introduce processing delays.

### A. Introducing Client-Side Delays

Turbo Intruder can introduce a short delay between requests.

However, this approach has an important limitation:

- Requests may be split across multiple TCP packets.
- The Single-Packet Attack technique cannot be used in the same way.
- On targets with high network jitter, the attack may become unreliable.

Therefore, client-side delays are not always suitable for synchronizing race condition requests.

### B. Triggering Server-Side Delays

Web servers and applications commonly implement rate limiting and resource-management mechanisms.

When too many requests arrive within a short period, the server may delay processing subsequent requests.

This behavior can potentially be abused to introduce a suitable server-side delay.

**General approach:**

1. Send a large number of harmless or dummy requests.
2. Intentionally trigger the application's rate or resource limit.
3. Cause subsequent requests to experience server-side processing delays.
4. Use the resulting delay to improve the alignment of the main concurrent requests.

This approach can make Single-Packet Attacks more practical in situations where delayed execution is required.

## 6. Tools and Techniques

| Tool / Technique | Purpose |
|---|---|
| Burp Suite Repeater | Sending and synchronizing multiple requests |
| Single-Packet Attack | Delivering concurrent requests with minimal timing differences |
| Turbo Intruder | Automating high-volume and precisely timed HTTP requests |
| Connection Warming | Reducing delays caused by back-end connection establishment |
| Rate Limit Analysis | Understanding server-side processing delays |

## 7. Key Takeaways

- Multi-endpoint race conditions involve concurrent requests targeting different application endpoints.
- They occur when multiple endpoints interact with shared application state without proper synchronization.
- Payment validation and order confirmation can create exploitable race windows.
- Network architecture and endpoint-specific processing can make race window alignment difficult.
- Connection warming helps reduce delays caused by back-end connection establishment.
- Turbo Intruder provides additional control over request timing and execution.
- Server-side rate or resource limits can potentially be abused to introduce processing delays.
- Developers should use atomic transactions, proper synchronization, and consistent state validation to prevent these vulnerabilities.

## 8. Prevention and Mitigation

Developers can reduce the risk of multi-endpoint race conditions by implementing:

- **Atomic Transactions:** Ensure critical operations, such as payment validation and order confirmation, execute as a single transaction.
- **Proper Synchronization:** Prevent concurrent requests from modifying shared application state inconsistently.
- **Server-Side State Validation:** Revalidate important application data immediately before committing an operation.
- **Database Locking:** Use appropriate locking mechanisms to prevent conflicting concurrent modifications.
- **Idempotency Controls:** Prevent duplicate or repeated requests from causing unintended state changes.
- **Consistent Workflow Validation:** Ensure that business operations follow the correct sequence and cannot be manipulated through concurrent requests.

---

**Related Topic:** [Race Condition Vulnerabilities](https://portswigger.net/web-security/race-conditions)

**Reference:** [PortSwigger Web Security Academy – Multi-Endpoint Race Conditions](https://portswigger.net/web-security/race-conditions#multi-endpoint-race-conditions)
