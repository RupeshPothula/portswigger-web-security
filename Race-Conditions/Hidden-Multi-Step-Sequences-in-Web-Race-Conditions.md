# Hidden Multi-Step Sequences in Web Race Conditions

## Overview

A hidden multi-step sequence occurs when a single HTTP request triggers several internal operations and the application passes through intermediate states before the request finishes. These intermediate states are called **sub-states**.

A race condition may occur when another request interacts with the same data while the first request is partway through its sequence. If the application temporarily exposes an unsafe state, the second request may observe or use it before the application reaches its intended final state.

This concept extends beyond simple limit overruns. It can affect authentication, account recovery, payments, workflow approvals, profile changes, and other operations involving shared state.

> **Important:** Only test systems you own or have explicit permission to assess.

## 1. Core Concepts and Key Terms

| Term | Meaning |
|---|---|
| Race condition | A flaw in which the result depends on the timing or interleaving of concurrent operations. |
| Collision | Two or more requests interact with the same record or shared state in a way that may cause interference. |
| Sub-state | A temporary intermediate state within a larger workflow. |
| Benchmark | The behavior observed when requests are sent under normal, non-concurrent conditions. |
| Second-order effect | A later observable consequence, such as an email, audit event, account change, or altered behavior on a subsequent page. |

## 2. Why Hidden Sub-States Matter

Consider a simplified multi-factor authentication (MFA) login flow:

```python
session["userid"] = user.id

if user.mfa_enabled:
    session["enforce_mfa"] = True
    # Generate and send an MFA code.
    # Redirect to the MFA entry page.
```

If the application sets the authenticated user identity before enabling or enforcing the MFA requirement, there may be a brief intermediate state in which the session appears authenticated but MFA enforcement is not yet active.

A concurrent request to a sensitive authenticated endpoint might arrive during this interval.

Whether this is exploitable depends on the application's actual session handling, request scheduling, and authorization checks. The pseudocode alone does not prove a vulnerability.

### Security Invariant

A session must not gain access to MFA-protected resources until the required MFA step has been successfully completed.

Authorization must be enforced server-side on every relevant request, rather than relying only on redirects or front-end navigation.

The same reasoning applies to other workflows: identify the intended security invariant, then investigate whether concurrency can expose a temporary violation of it.

## 3. The Methodology: Predict, Probe, Prove

A structured approach helps identify and verify potential race conditions.

1. **Predict:** Identify security-critical endpoints and determine which requests could collide on the same record or shared state.
2. **Probe:** Establish normal behavior, then compare it with carefully synchronized concurrent requests.
3. **Prove:** Reduce the test to the smallest repeatable case and demonstrate the security-relevant impact.

Avoid testing every endpoint indiscriminately. Prioritize high-impact operations and plausible shared-state interactions.

## 4. Phase 1 — Predict Potential Collisions

### 4.1 Map the Application

During authorized reconnaissance, map relevant workflows and record:

- Endpoints and HTTP methods.
- Authentication and authorization requirements.
- Records or state each endpoint reads or modifies.
- Session, token, and MFA transitions.
- Related background jobs, email/SMS notifications, and asynchronous processing.
- Whether multiple requests can act on the same account, order, reset token, transaction, or other object.

### 4.2 Ask Two Questions for Each Candidate

**A. Is this endpoint security-critical?**

Prioritize endpoints that affect:

- Login, MFA, session creation, or privilege changes.
- Password reset, account recovery, or email/phone verification.
- Payments, purchases, balances, coupons, inventory, or quotas.
- Permission changes, approvals, invitations, or account ownership.
- Sensitive records or actions protected by authorization decisions.

**B. Is there collision potential?**

Look for two or more requests that can affect the same record or shared state. Similar-looking requests that operate on entirely separate records may not create the relevant collision.

Examples:

- **Low apparent collision potential:** Two password-reset requests affect unrelated user records.
- **Higher collision potential:** Two requests update the same account's reset state, token, or verification status.

The second case is worth investigating because the operations may interleave over shared state. It is a candidate for testing, not proof of a vulnerability.

### 4.3 Build a Collision Matrix

| Request A | Request B | Shared State | Why It May Matter |
|---|---|---|---|
| Login/MFA initiation | Sensitive authenticated action | Session and MFA state | A temporary authentication-state inconsistency |
| Password-reset initiation | Another reset-related request | Reset token/status for one account | Token replacement or inconsistent reset state |
| Apply discount | Submit order | Order and discount usage | Discount validation and order creation may interleave |
| Change permissions | Access protected resource | User roles and authorization state | A stale or partially updated authorization decision |
| Redeem benefit | Redeem the same benefit again | Redemption record or quota | Duplicate or inconsistent state transition |

Adapt these examples to the application's actual design. Do not assume a request pair is vulnerable simply because it appears in the table.

### 4.4 Record a Hypothesis Before Testing

For each candidate, document:

- **Expected invariant:** What must always remain true?
- **Request A:** The operation that may create an intermediate state.
- **Request B:** The operation that may observe or exploit that state.
- **Shared object/state:** The account, record, token, order, or session involved.
- **Expected normal result:** What should happen sequentially?
- **Potential concurrent deviation:** What observation would indicate a problem?

A clear hypothesis helps distinguish a genuine race condition from ordinary application behavior.

## 5. Phase 2 — Probe for Clues

The goal is to compare normal behavior with concurrent behavior while controlling avoidable sources of noise.

### 5.1 Establish a Baseline

Before sending requests concurrently:

1. Use a controlled test account and known test data.
2. Capture the relevant requests and responses.
3. Send candidate requests sequentially.
4. Record status codes, response bodies, redirects, timing, cookies, and relevant headers.
5. Check the resulting application state.
6. Inspect second-order effects, such as test emails, notifications, audit logs, or transaction records.
7. Repeat the baseline if behavior is inconsistent.

In Burp Repeater, requests can be grouped and sent using **Send group in sequence (separate connections)** to establish sequential behavior.

Keep requests, session state, and test data consistent between baseline and concurrent runs. Otherwise, the comparison may be misleading.

### 5.2 Send Requests Concurrently

Once baseline behavior is understood:

1. Group the candidate requests in Burp Repeater.
2. Use **Send group in parallel** to compare concurrent behavior with the sequential baseline.
3. Where supported and appropriate, use synchronization techniques that reduce network jitter, such as Burp's single-packet attack.
4. If HTTP/2-based synchronization is unavailable, consider the documented last-byte synchronization approach where supported.
5. Repeat controlled tests to determine whether the result is consistent.
6. Compare immediate responses and the final server-side state.

See the official documentation:

- [Burp Repeater — Sending requests in sequence](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-sequence)
- [Burp Repeater — Sending requests in parallel](https://portswigger.net/burp/documentation/desktop/tools/repeater/send-group#sending-requests-in-parallel)

An alternative for authorized testing is [Turbo Intruder](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988), a Burp extension for sending and analyzing large numbers of requests. Use the minimum request volume needed and avoid unnecessary load.

### 5.3 Look for Deviations

Potential clues include:

- Different status codes, response bodies, or redirects.
- An unexpected success or authorization result.
- A response reflecting inconsistent or stale state.
- A changed cookie, token, session property, or workflow state.
- Duplicate or missing records.
- An unexpected email, notification, audit event, or transaction.
- A persistent change in application behavior after requests complete.

A different response is only a clue. It may result from normal nondeterminism, rate limiting, caching, network errors, or test setup.

Verify the effect against the expected security invariant before concluding that a vulnerability exists.

### 5.4 Optional Burp Professional Aid

Burp Professional users may be able to use PortSwigger's **Trigger race conditions** custom action to send parallel requests from Repeater.

Availability and behavior depend on the Burp version and configuration. Consult the relevant documentation:

- [Burp Suite — Custom actions](https://portswigger.net/burp/documentation/desktop/tools/repeater/http-messages/custom-actions)
- [Probe for Race Condition — Bambda example](https://github.com/PortSwigger/bambdas/blob/main/CustomAction/ProbeForRaceCondition.bambda)

A helper can speed up testing, but it does not replace a baseline, state inspection, or proof of impact.

## 6. Phase 3 — Prove the Concept

A convincing finding requires more than one surprising response.

### 6.1 Minimize the Test

1. Identify which requests are necessary to reproduce the behavior.
2. Remove unrelated requests and unnecessary variables.
3. Keep the account, object, session, and preconditions controlled.
4. Repeat the smallest test sequentially and concurrently.
5. Confirm that the behavior is tied to concurrency rather than a different request body, session, token, or test state.

### 6.2 Verify the Security Invariant

Check whether concurrent execution violates the invariant identified during the prediction phase.

Examples:

- Was a protected action accepted before MFA completion?
- Was a one-time token accepted more than once?
- Was an authorization decision made using an inconsistent state?
- Did a transaction or redemption occur more times than allowed?

Use safe, observable proof in a test environment. Do not infer impact solely from timing differences or an unusual HTTP response.

### 6.3 Confirm Repeatability and Impact

- Repeat the minimal case to estimate reliability.
- Record successful and unsuccessful attempts.
- Verify the final server-side state where possible.
- Determine which security boundary or business rule was crossed.
- Stop once sufficient evidence exists; avoid unnecessary load or destructive actions.

### 6.4 Think Structurally

Treat a race condition as a possible structural weakness in state transitions, not just an isolated endpoint bug.

Ask why the intermediate state existed and whether related workflows share the same design flaw.

For example, if authorization depends on multiple fields being updated together, determine whether the application enforces a safe state transition atomically and whether every sensitive endpoint checks the required conditions.

## 7. Burp Suite Testing Workflow

A repeatable high-level workflow is:

1. **Capture:** Intercept relevant requests while performing the workflow normally.
2. **Prepare:** Send the requests to Repeater and maintain a controlled test session.
3. **Sequence:** Group the requests and send them sequentially to establish a baseline.
4. **Record:** Save responses and inspect the resulting application state.
5. **Parallelize:** Send the same candidate requests in parallel using an appropriate synchronization option.
6. **Compare:** Review response differences and side effects.
7. **Reduce:** Remove unnecessary requests until the smallest reproducible case remains.
8. **Repeat:** Re-run the baseline and concurrent test to assess reliability.
9. **Document:** Preserve sanitized requests, timestamps, responses, and state evidence.

Use the option names available in your installed Burp version because the interface may change.

## 8. What to Observe

Use a consistent observation record for each test run.

| Observation | What to Record |
|---|---|
| Test identifier | Unique label for the test run |
| Preconditions | Account state, roles, MFA status, tokens, and object state |
| Execution mode | Sequential or concurrent; synchronization method |
| Request pair/group | Endpoint and method, with secrets removed |
| Response evidence | Status, relevant headers, body differences, and redirects |
| Timing | Start/end timestamps if available; timing alone is not proof |
| Side effects | Emails, notifications, audit events, and created or modified records |
| Final state | Server-side state after all requests finish |
| Repeatability | Number of attempts and observed deviations |
| Invariant result | Whether the expected security rule held |
| Conclusion | No issue observed, inconclusive, or reproducible finding |

### Response Comparison Tips

- Compare semantically relevant fields, not just raw response length.
- Redact session cookies, authorization headers, passwords, MFA codes, reset tokens, and personal data from saved evidence.
- Check whether redirects are hiding a server-side state change.
- Where possible, inspect final state through a trusted test interface or server-side logs.
- Keep test data unique enough to distinguish one run from another.

## 9. Common Pitfalls

- **Testing unrelated records:** Requests may not collide if they never touch shared state.
- **Skipping the baseline:** Without normal behavior, a deviation is difficult to interpret.
- **Changing multiple variables:** A different session or token can explain the result.
- **Relying on one attempt:** Race behavior can be intermittent.
- **Treating timing as proof:** A fast response does not establish security impact.
- **Ignoring side effects:** The response may look normal while the final state or a notification differs.
- **Overloading the target:** Excessive parallel traffic can disrupt service and obscure the result.
- **Assuming a redirect enforces security:** Server-side authorization must protect the sensitive operation itself.
- **Stopping at discovery:** A clue is not a confirmed vulnerability until the relevant invariant violation is demonstrated.
- **Overstating the result:** Report only the impact supported by repeatable evidence.

## 10. Test Worksheet and Checklist

Copy this section for each candidate workflow.

### Candidate Checklist

- [ ] Application and environment documented.
- [ ] Permission to test confirmed.
- [ ] Test account and non-production data prepared.
- [ ] Workflow and relevant endpoints mapped.
- [ ] Security-critical operation identified.
- [ ] Shared record or state identified.
- [ ] Expected security invariant written down.
- [ ] Sequential baseline captured.
- [ ] Concurrent test run under controlled conditions.
- [ ] Responses and final state compared.
- [ ] Second-order effects checked where relevant.
- [ ] Test repeated to assess reliability.
- [ ] Minimal reproduction isolated.
- [ ] Secrets and personal data redacted from evidence.
- [ ] Impact and limitations documented.
- [ ] Testing stopped once sufficient proof was collected.

### Result Summary

```text
Hypothesis:
Expected invariant:
Request A:
Request B:
Shared state:
Sequential result:
Concurrent result:
Final-state evidence:
Repeatability:
Security impact:
Conclusion: No issue observed / Inconclusive / Reproducible finding
```

## 11. Reporting Findings

A useful report should include:

1. **Title:** State the workflow and race-condition class.
2. **Summary:** Explain the violated invariant in plain language.
3. **Affected component:** Identify the relevant endpoint(s) and workflow.
4. **Preconditions:** Describe the account state, permissions, and required setup.
5. **Reproduction:** Provide the smallest sequential baseline and concurrent test procedure.
6. **Evidence:** Include sanitized requests/responses and final-state or side-effect proof.
7. **Reliability:** State how often the behavior reproduced and any limitations.
8. **Impact:** Describe the demonstrated security or business-rule consequence without speculation.
9. **Remediation direction:** Explain which state transition or authorization check must be made safe.

## 12. Defensive Remediation

The correct fix depends on the root cause. Common approaches include:

- **Enforce authorization and MFA completion:** Check these requirements on every sensitive server-side operation.
- **Avoid partially authenticated sessions:** Represent authentication state explicitly and do not grant protected access before MFA succeeds.
- **Use atomic state transitions:** Ensure security-critical state changes cannot be observed in an unsafe intermediate state.
- **Apply appropriate database protections:** Use transactions, suitable locking, uniqueness constraints, or compare-and-swap/version checks when appropriate for the data model.
- **Enforce single-use operations:** Ensure one-time tokens and operations remain single-use at the data layer.
- **Protect validation and state changes:** Prevent a security check from being separated from the consequential update by an unsafe race window.
- **Add concurrency-focused regression tests:** Test the security invariant, not only one observed request sequence.

A UI redirect or client-side check is not a substitute for server-side enforcement.

## Main Takeaway

Hidden multi-step sequences create opportunities for race conditions when concurrent requests interact with temporary application states before a workflow reaches its intended final state.

The **Predict, Probe, Prove** methodology helps identify plausible collisions, compare sequential and concurrent behavior, and establish reproducible security impact.

The most important step is to verify whether a concurrency issue violates a specific security invariant. A surprising response or timing difference alone is not sufficient evidence.

**For prevention, applications must enforce authorization and critical state transitions safely on the server side, using appropriate atomic operations, transaction handling, and concurrency controls.**
