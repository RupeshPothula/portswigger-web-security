# Limit Overrun Race Conditions

## 1. Introduction

A **limit overrun race condition** occurs when an application allows a user to exceed a restriction imposed by its business logic because multiple requests are processed concurrently.

Common examples include single-use discount codes, gift cards, account balance checks, CAPTCHA solutions, and rate limits.

## 2. How Limit Overrun Race Conditions Work

Consider an online store that provides a one-time discount code. The application performs the following operations:

1. Check whether the user has already used the discount code.
2. Apply the discount to the order total.
3. Update the database to record that the code has been used.

Under normal conditions, a subsequent request should fail because the code has already been redeemed.

However, if two requests arrive almost simultaneously, the following sequence may occur:

1. Request A checks whether the code has been used and finds that it has not.
2. Request B performs the same check before Request A updates the database and also finds that the code has not been used.
3. Both requests apply the discount.
4. The application records that the code has been used, but the discount may already have been applied twice.

This allows the user to exceed the intended one-time usage restriction.

### Temporary Sub-state and Race Window

During request processing, the application passes through a **temporary sub-state** in which the discount has been validated but the database has not yet recorded its use.

The race window begins when the application enters this vulnerable intermediate state and ends when the database update prevents further successful use.

If another request reaches the same state during this window, both requests may proceed based on the same outdated information.

The race window may last only milliseconds, making precise request timing important.

## 3. Common Examples

Limit overrun race conditions can affect several types of functionality:

- **Gift cards:** Redeeming the same single-use gift card multiple times.
- **Product ratings:** Submitting multiple ratings when only one should be allowed.
- **Banking operations:** Withdrawing or transferring more money than the available account balance.
- **CAPTCHA validation:** Reusing a single CAPTCHA solution for multiple operations.
- **Rate limiting:** Sending more requests than an application intends to permit, potentially bypassing anti-brute-force protection.

The impact depends on the application's business logic and the operation being exploited.

## 4. Connection to TOCTOU Vulnerabilities

Limit overrun race conditions are a subtype of **Time-of-Check to Time-of-Use (TOCTOU)** vulnerabilities.

A TOCTOU flaw occurs when an application checks a condition and then uses the result later, allowing the relevant state to change between the check and the operation.

For example:

1. **Time of check:** The application verifies that an account has sufficient funds.
2. **Time of use:** The application processes a withdrawal.
3. **Race condition:** Two withdrawals pass the balance check before either updates the account balance.

Both withdrawals may be approved based on the same original balance, potentially resulting in an overdraft.

The underlying issue is that the security-sensitive check and the operation are not adequately protected against concurrent requests.

## 5. Detecting and Exploiting Limit Overruns with Burp Repeater

### Basic Methodology

The general process for identifying a potential limit overrun is:

1. Identify a single-use or rate-limited endpoint with a meaningful security or business impact.
2. Understand the normal request and response behavior.
3. Send multiple requests in quick succession using Burp Suite Repeater.
4. Examine the responses and resulting application state to determine whether the restriction was exceeded.

The main challenge is aligning the requests so that their processing overlaps during the race window.

### Why Timing Matters

Even if requests are sent at nearly the same time, they may reach and execute on the server at different times because of:

- Network latency and jitter.
- Server-side processing delays.
- Database response times.
- Request scheduling and execution order.

These unpredictable factors make it difficult to trigger a race condition reliably.

### Parallel Requests in Burp Repeater

Burp Suite 2023.9 introduced enhanced capabilities for sending groups of requests in parallel through Repeater. These features help reduce the effect of network jitter and improve the chances of overlapping requests.

Burp automatically adapts its synchronization technique to the HTTP version supported by the server.

| HTTP version | Synchronization technique |
|---|---|
| HTTP/1 | Last-byte synchronization |
| HTTP/2 | Single-packet attack |

#### Last-Byte Synchronization

For HTTP/1, the technique coordinates the final byte of multiple requests so that they are more likely to reach the server at approximately the same time.

This helps reduce timing differences introduced by the network.

#### Single-Packet Attack

For HTTP/2, Burp can use the **single-packet attack**, a technique introduced by PortSwigger Research and presented at Black Hat USA 2023.

It enables multiple requests to be completed using a single TCP packet, allowing approximately 20–30 requests to be sent simultaneously in suitable conditions.

Because the requests share a tightly synchronized transmission, this technique can substantially reduce interference from network jitter.

### Why Send Multiple Requests?

Although two overlapping requests may be sufficient to trigger a race condition, sending a larger group of requests can improve the chances of success during initial testing.

This is useful because of **server-side jitter**: unpredictable delays caused by internal processing, scheduling, or database operations.

The objective is to determine whether concurrent requests can bypass the application's intended restriction, not simply whether multiple requests can be transmitted together.

## 6. Detecting and Exploiting Limit Overruns with Turbo Intruder

**Turbo Intruder** is a Burp Suite extension designed for sending large numbers of HTTP requests and handling more complex request-timing scenarios.

It supports the single-packet attack technique and is useful when testing requires:

- Multiple retries.
- Staggered request timing.
- A very large number of requests.
- More control over concurrency and request scheduling.

Turbo Intruder requires some familiarity with Python.

### Requirements for the Single-Packet Attack

To use this technique with Turbo Intruder:

1. Ensure the target supports HTTP/2. The single-packet attack is incompatible with HTTP/1.
2. Configure the request engine with `engine=Engine.BURP2`.
3. Set `concurrentConnections=1`.
4. Group requests using a named gate through the `gate` argument of `engine.queue()`.
5. Open the corresponding gate with `engine.openGate()` to release the queued requests together.

### Example

The following example demonstrates the basic structure of a Turbo Intruder script for an authorized test environment:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        engine=Engine.BURP2
    )

    # Queue 20 requests in gate '1'
    for i in range(20):
        engine.queue(target.req, gate='1')

    # Release the queued requests together
    engine.openGate('1')
```

**How it works:**

- `RequestEngine` configures the request engine.
- `Engine.BURP2` enables the HTTP/2 engine.
- `concurrentConnections=1` uses a single connection for the technique.
- `engine.queue()` queues requests under the named gate.
- `engine.openGate()` releases the requests in that group together.

The example illustrates synchronized request delivery; whether a vulnerability is triggered still depends on the application's processing and the race window.

For the full implementation, refer to the `race-single-packet-attack.py` template included in Turbo Intruder's default examples directory.

## 7. Key Takeaways

- Limit overrun race conditions exploit restrictions enforced by application business logic.
- They commonly occur when multiple requests pass the same validation check before the application updates its state.
- A race window is the brief period during which overlapping requests can cause an unintended result.
- Limit overruns are a subtype of TOCTOU vulnerabilities.
- Burp Repeater supports parallel request groups and adapts synchronization to HTTP/1 or HTTP/2.
- The single-packet attack helps reduce network jitter when testing HTTP/2 applications.
- Turbo Intruder is useful for more complex tests involving retries, custom timing, or large request groups.
- A successful test must verify that the intended business restriction was actually bypassed.

## Further Reading

- [PortSwigger — Limit Over run race conditions](https://portswigger.net/web-security/race-conditions#limit-overrun-race-conditions)

