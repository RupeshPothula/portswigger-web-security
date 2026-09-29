# Vulnerabilities in Password-Based Login

## Introduction

In password-based authentication, a user authenticates by providing a username and a secret password.

The security of this authentication method depends heavily on keeping the credentials secret.

If an attacker can **guess or obtain another user's credentials**, they may be able to access that user's account and all of the functionality available to it.

Common vulnerabilities in password-based login include:

* Brute-force attacks
* Weak username and password choices
* Username enumeration
* Flawed brute-force protection
* Account locking weaknesses
* User rate-limiting weaknesses
* Vulnerabilities in HTTP Basic Authentication

---

# 1. Brute-Force Attacks

A **brute-force attack** is a trial-and-error process used to discover valid login credentials.

Instead of manually trying credentials, attackers can automate large numbers of login attempts using lists of:

```text
Usernames
Passwords
Username + password combinations
```

A simple concept is:

```text
Candidate username
        +
Candidate password
        ↓
Login request
        ↓
Observe response
        ↓
Valid / Invalid
```

Brute-force attacks become more effective when the application does not have sufficient protection against repeated login attempts.

---

# 2. Brute-Forcing Usernames

Usernames can sometimes be easier to guess than passwords because applications often use predictable naming conventions.

For example, an organization may use:

```text
firstname.lastname@company.com
```

or common administrative usernames such as:

```text
admin
administrator
```

During security testing, publicly available information can sometimes reveal possible usernames.

Potential sources include:

* Public user profiles
* Email addresses exposed in HTTP responses
* Predictable username formats
* Other application functionality that reveals usernames

Finding valid usernames can significantly reduce the amount of guessing required during a password attack.

---

# 3. Brute-Forcing Passwords

Passwords can also be guessed using wordlists and automated requests.

The difficulty depends partly on the strength and unpredictability of the password.

Websites may enforce password policies such as:

* Minimum password length
* Uppercase and lowercase characters
* Special characters

However, password policies do not necessarily result in unpredictable passwords.

Users often modify familiar passwords to satisfy password requirements.

For example:

```text
mypassword
     ↓
Mypassword1!
```

Similarly, when users are required to change passwords regularly, they may make predictable modifications:

```text
Mypassword1!
        ↓
Mypassword2!
```

or:

```text
Mypassword1!
        ↓
Mypassword1?
```

Therefore, brute-force attacks can use knowledge about common human password choices rather than trying every possible character combination.

---

# 4. Username Enumeration

**Username enumeration** occurs when an application's behavior allows an attacker to determine whether a username is valid.

For example, suppose the login page responds differently to:

```text
Invalid username
```

and:

```text
Incorrect password
```

The difference can reveal whether the submitted username exists.

Username enumeration can occur in several places, including:

* Login pages
* Registration forms
* Account recovery functionality
* Other application responses

For an attacker, discovering valid usernames is valuable because it reduces the search space for password guessing.

---

## Indicators of Username Enumeration

When testing a login system, compare the responses for valid and invalid usernames.

Important differences can include:

### HTTP Status Codes

Most incorrect login attempts may return the same status code.

If one username produces a different status code, it may indicate that the username is valid.

---

### Error Messages

An application may accidentally return different messages depending on what went wrong.

For example:

```text
Invalid username or password
```

versus:

```text
Incorrect password
```

Even very small differences can reveal information.

---

### Response Timing

The time taken to process different login attempts can sometimes provide clues.

For example, the application might:

```text
Check username
      ↓
If username exists
      ↓
Check password
```

An invalid username may therefore follow a different processing path from a valid username.

Even a small timing difference can potentially be detected when many requests are compared.

---

# 5. Flawed Brute-Force Protection

Websites commonly attempt to slow down or prevent brute-force attacks.

Two common approaches are:

```text
Account locking
User/IP rate limiting
```

However, flawed implementations can sometimes be bypassed.

---

# 6. Account Locking

Account locking temporarily prevents login attempts against an account after a certain number of failed attempts.

For example:

```text
Attempt 1 → Failed
Attempt 2 → Failed
Attempt 3 → Failed
             ↓
       Account locked
```

This can make directly brute-forcing one specific account more difficult.

However, account locking has weaknesses.

---

## Username Enumeration Through Account Locking

The application's response when an account becomes locked can itself reveal whether a username exists.

For example:

```text
Valid username
      ↓
Repeated failed attempts
      ↓
Account locked
```

while an invalid username may never produce the same behavior.

Therefore, account locking can unintentionally help with username enumeration.

---

## Brute-Forcing Across Multiple Accounts

Account locking is primarily designed to protect an individual account.

An attacker may instead distribute a small number of password guesses across many candidate usernames.

For example:

```text
User 1 → Password A
User 2 → Password A
User 3 → Password A
...
```

Then:

```text
User 1 → Password B
User 2 → Password B
User 3 → Password B
...
```

The goal is to avoid exceeding the lockout threshold for any individual account.

This demonstrates that account locking alone is not necessarily sufficient protection against brute-force attacks.

---

# 7. Credential Stuffing

**Credential stuffing** is different from traditional password brute-forcing.

Instead of generating password guesses, an attacker uses previously leaked username/password combinations obtained from other sources.

For example:

```text
username1:password1
username2:password2
username3:password3
```

The attack relies on **password reuse**.

If a user uses the same credentials on multiple websites, credentials compromised from one website may also work on another.

---

## Why Account Locking May Not Stop Credential Stuffing

In credential stuffing, an attacker may try only one known password against each username.

For example:

```text
User A → known password
User B → known password
User C → known password
User D → known password
```

Because each account may only receive one login attempt, an account-locking threshold may never be reached.

This means credential stuffing requires separate defensive considerations.

---

# 8. User/IP Rate Limiting

Another protection mechanism is **rate limiting**.

The application limits the number of login attempts that can be made within a particular period.

For example:

```text
Too many login attempts
        ↓
Requests temporarily blocked
```

The block may be removed:

* Automatically after a period of time
* By an administrator
* After completing a CAPTCHA or another verification mechanism

Rate limiting can be preferable to account locking in some situations because it does not directly lock a specific user's account.

However, poorly implemented rate limiting can still have weaknesses.

---

# 9. Weak Rate-Limiting Logic

A rate limit may be based on the apparent source of requests, such as an IP address.

If the application does not correctly determine or enforce the source of requests, an attacker may potentially manipulate how their requests are identified.

Another possible weakness is when an application allows multiple password guesses to be included in a single request.

For example, instead of:

```text
Request 1 → Password A
Request 2 → Password B
Request 3 → Password C
```

the application might incorrectly accept multiple password values within one request.

This can undermine protections that count the number of HTTP requests rather than the number of actual password guesses.

---

# 10. HTTP Basic Authentication

**HTTP Basic Authentication** is an older authentication mechanism that may still be encountered because it is relatively simple to implement.

The basic process is:

```text
Username + Password
        ↓
Combine
        ↓
Base64 encode
        ↓
Authorization header
```

The resulting request contains an HTTP header such as:

```http
Authorization: Basic base64(username:password)
```

The browser can automatically include this header with subsequent requests.

---

# 11. Weaknesses of HTTP Basic Authentication

HTTP Basic Authentication has several security limitations.

## Credentials Are Sent Repeatedly

The credentials are represented in the authorization header and are sent with requests.

If the connection is not adequately protected, an attacker positioned to observe the traffic may potentially capture the credentials.

HTTPS and appropriate transport-security protections are therefore important.

---

## Base64 Is Not Encryption

An important point is:

```text
Base64 ≠ Encryption
```

Base64 is an encoding mechanism.

It does not make the credentials secret by itself.

For example:

```text
username:password
       ↓
Base64 encoding
       ↓
Encoded value
```

The encoded value can be decoded back into the original credentials.

Therefore, Base64 should not be treated as a security mechanism.

---

## Weak Brute-Force Protection

HTTP Basic Authentication implementations may lack effective brute-force protection.

Because the authorization value is based on static credentials, repeated credential guessing may be possible if appropriate protections are not implemented.

---

## Session-Related Attacks

HTTP Basic Authentication also does not inherently provide protection against some session-related attacks, including **CSRF**.

Therefore, relying on Basic Authentication alone does not automatically provide protection against these attacks.

---

# 12. Why a Low-Privilege Account Can Still Matter

Compromising an account does not necessarily mean the attacker immediately gains administrative privileges.

However, even a low-privileged account can provide:

```text
Authenticated access
       ↓
Additional pages
       ↓
Additional functionality
       ↓
Larger attack surface
```

The compromised credentials may also be reused in other contexts if users have reused passwords.

Therefore, the impact of authentication weaknesses should not be judged only by the privilege level of the initially compromised account.

---

# Overall Testing Mindset

When assessing password-based authentication, think about the entire authentication process:

```text
             LOGIN
               │
       ┌───────┴───────┐
       ↓               ↓
   Username         Password
       │               │
       ↓               ↓
 Enumeration       Brute-force
       │               │
       └───────┬───────┘
               ↓
      Brute-force protection
               │
       ┌───────┴────────┐
       ↓                ↓
 Account locking    Rate limiting
       │                │
       └───────┬────────┘
               ↓
      Authentication result
```

The goal is not just to test whether a password can be guessed. You should also understand whether the application leaks information about valid usernames and whether its protections are implemented correctly.

---

# Key Takeaways

* Password-based authentication relies on the secrecy of user credentials.
* Brute-force attacks use repeated credential guesses to find valid credentials.
* Predictable usernames can make username discovery easier.
* Human password habits can make supposedly complex passwords more predictable.
* Username enumeration occurs when application behavior reveals whether a username is valid.
* Differences in status codes, error messages, or response timing can sometimes reveal valid usernames.
* Account locking can slow targeted brute-force attacks but can also introduce username-enumeration issues.
* Rate limiting can restrict repeated login attempts but must be implemented carefully.
* Credential stuffing uses previously compromised username/password combinations and relies heavily on password reuse.
* HTTP Basic Authentication uses Base64 encoding of credentials; Base64 does not provide encryption.
* Basic Authentication can be vulnerable to credential interception if transport security is inadequate and may lack effective brute-force protection.
* Even a low-privileged compromised account can expose additional application functionality and attack surface.
