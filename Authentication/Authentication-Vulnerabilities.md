# Authentication Vulnerabilities

## Introduction

Authentication is one of the most important security mechanisms in a web application because it determines whether a user is who they claim to be.

Authentication vulnerabilities can allow attackers to gain unauthorized access to:

* Sensitive data
* User accounts
* Application functionality
* Privileged functionality

Authentication vulnerabilities can also provide additional attack surface that may be used for further attacks.

Because of this, it is important to understand:

* Common authentication mechanisms
* Vulnerabilities in these mechanisms
* Inherent weaknesses in different authentication mechanisms
* Vulnerabilities caused by poor implementation
* Methods for making authentication mechanisms more robust

---

# What is Authentication?

**Authentication** is the process of verifying the identity of a user or client.

For example, when a user logs into a website using:

```text
Username: Carlos123
Password: ********
```

the application verifies whether the supplied credentials belong to the claimed user.

A website can potentially be accessed by anyone connected to the internet, so strong authentication mechanisms are an important part of web security.

---

# Types of Authentication Factors

Authentication can be based on three main types of factors.

## 1. Something You Know

This is information that the user knows.

Examples include:

* Password
* PIN
* Answer to a security question

These are known as **knowledge factors**.

Example:

```text
Username + Password
```

The user proves their identity by providing information that should only be known to them.

---

## 2. Something You Have

This is a physical object or device that the user possesses.

Examples include:

* Mobile phone
* Security token

These are known as **possession factors**.

For example, a website may send a verification code to a user's mobile phone.

The user then proves that they possess the device by providing the code.

---

## 3. Something You Are or Do

This factor is based on characteristics of the user.

Examples include:

* Biometrics
* Patterns of behavior

These are known as **inherence factors**.

Biometric authentication can include characteristics that are associated with the individual user.

---

# Authentication vs Authorization

Authentication and authorization are related but represent different concepts.

## Authentication

Authentication answers:

> **"Who are you?"**

It verifies that a user is actually the person they claim to be.

For example:

```text
User claims:
Carlos123

Authentication:
Is this really Carlos123?
```

---

## Authorization

Authorization answers:

> **"What are you allowed to do?"**

After a user has been authenticated, the application determines what that user is permitted to access or perform.

For example:

```text
Authentication
      ↓
User is Carlos123
      ↓
Authorization
      ↓
What can Carlos123 access?
```

A user may be authenticated successfully but still not be authorized to perform certain actions.

---

## Simple Example

Suppose `Carlos123` successfully logs into an application.

Authentication determines:

```text
Carlos123 is a valid authenticated user.
```

Authorization then determines whether Carlos123 can:

```text
View personal information
Delete another user's account
Access administrative functionality
```

Therefore:

```text
Authentication → Identity
Authorization  → Permissions
```

---

# How Authentication Vulnerabilities Arise

Authentication vulnerabilities commonly occur in two ways.

## 1. Weak Authentication Mechanisms

An authentication mechanism may be weak because it does not adequately protect against **brute-force attacks**.

For example, if an application does not properly restrict repeated login attempts, an attacker may be able to repeatedly try different credentials.

This can potentially result in unauthorized account access.

---

## 2. Logic Flaws and Poor Implementation

Authentication can also contain **logic flaws** or implementation mistakes.

These weaknesses may allow an attacker to bypass the authentication mechanism entirely.

This is sometimes referred to as **broken authentication**.

For example, if the application's authentication logic does not correctly verify the user's identity, an attacker may be able to access functionality without successfully completing the intended authentication process.

---

# Why Authentication Logic Flaws Are Important

Logic flaws can cause applications to behave in unexpected ways.

In many parts of web development, unexpected behavior may not necessarily result in a security vulnerability.

Authentication is different because authentication directly controls access to the application.

Therefore:

```text
Authentication logic flaw
          ↓
Authentication bypass
          ↓
Unauthorized access
```

A small mistake in authentication logic can therefore have significant security consequences.

---

# Impact of Authentication Vulnerabilities

The impact depends on which account or functionality is compromised.

## Compromising Another User's Account

If an attacker bypasses authentication or successfully brute-forces another user's credentials, they may gain access to everything that the compromised account is allowed to access.

This could include:

* User data
* Application functionality
* Additional pages
* Other resources available to that account

---

## Compromising a High-Privileged Account

If an attacker compromises a highly privileged account, such as a system administrator, the impact can be much greater.

An administrator may have access to:

```text
Application administration
        ↓
Sensitive application data
        ↓
Additional functionality
        ↓
Potential internal infrastructure
```

Therefore, compromising a high-privileged account can potentially lead to control over a large portion of the application and access to internal infrastructure.

---

## Compromising a Low-Privileged Account

A low-privileged account can still be valuable to an attacker.

Even if the account does not have administrative privileges, it may provide access to:

* Data that should not be accessible to the attacker
* Commercially sensitive information
* Additional application pages
* Functionality that is not publicly accessible

This can provide an attacker with additional attack surface for further attacks.

---

# Public vs Internal Attack Surface

An important concept is that not all functionality may be available from publicly accessible pages.

For example:

```text
Public pages
     ↓
Limited functionality

Authenticated pages
     ↓
Additional functionality

Privileged pages
     ↓
Highly sensitive functionality
```

Some high-severity attacks may not be possible from publicly accessible pages but may become possible after obtaining access to an internal or authenticated page.

Therefore, authentication vulnerabilities can have an impact beyond simply gaining access to one account.

---

# Authentication Attack Surface

The overall authentication attack surface can include:

```text
Login functionality
       ↓
Credential verification
       ↓
Authentication logic
       ↓
Account access
       ↓
Authenticated functionality
       ↓
Privileged functionality
```

A weakness at any stage can potentially affect the security of the application.

---

# Key Takeaways

* Authentication verifies the identity of a user or client.
* Authentication is a critical part of web application security.
* Authentication can be based on something you know, have, or are/do.
* Authentication and authorization are different:

  * **Authentication → Who are you?**
  * **Authorization → What are you allowed to do?**
* Authentication vulnerabilities can arise from weak protection against brute-force attacks.
* Logic flaws and poor implementation can result in authentication bypasses.
* Compromising a user's account provides access to the data and functionality available to that account.
* Compromising a high-privileged account can potentially provide much greater access.
* Low-privileged accounts can still expose sensitive information and additional attack surface.
* Authentication vulnerabilities can therefore affect much more than the login page itself.
