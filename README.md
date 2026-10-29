# PortSwigger Web Security Academy — Learning Notes

<p align="center">
  <img src="https://portswigger.net/content/images/logos/portswigger-logo.svg" alt="PortSwigger Logo" width="300"/>
</p>

<p align="center">
  <b>Web Application Security | Vulnerability Research | Hands-on Labs</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Focus-Web%20Security-red?style=for-the-badge" alt="Web Security"/>
  <img src="https://img.shields.io/badge/Platform-PortSwigger%20Academy-orange?style=for-the-badge" alt="PortSwigger Academy"/>
  <img src="https://img.shields.io/badge/Notes-Markdown-blue?style=for-the-badge" alt="Markdown Notes"/>
</p>

## About This Repository

Welcome to my **PortSwigger Web Security Academy Learning Repository**!

This repository contains my personal notes, summarized concepts, vulnerability analysis, testing methodologies, and practical learnings from the [PortSwigger Web Security Academy](https://portswigger.net/web-security).

The goal is to build a structured reference for understanding web application vulnerabilities, learning how they can be identified and tested in authorized environments, and understanding the techniques used to prevent them.

I document the modules I study, keeping the notes concise, technically useful, and easy to revisit during cybersecurity preparation and revision.

## Learning Objectives

- Understand common web application vulnerabilities and their underlying causes.
- Learn practical web security testing techniques using Burp Suite.
- Understand how to identify, analyze, and validate security weaknesses.
- Study authentication, authorization, session management, and access control.
- Explore server-side and client-side attack surfaces.
- Understand vulnerability impact, remediation strategies, and secure coding practices.
- Maintain organized notes for future reference and interview preparation.

## Topics Covered

The following sections represent the major learning areas of PortSwigger Web Security Academy. Individual folders and notes are added as I progress through the modules.

### 1. Server-Side Vulnerabilities

| Topic | Key Areas |
|---|---|
| SQL Injection | Database queries, UNION attacks, blind SQL injection |
| Authentication | Password-based login, brute force, MFA, account recovery |
| Path Traversal | Accessing files outside intended directories |
| OS Command Injection | Server-side command execution |
| Business Logic Vulnerabilities | Logic flaws, workflow bypasses, limit overrun |
| Information Disclosure | Exposed data, error messages, debug information |
| Access Control | IDOR, privilege escalation, authorization bypasses |
| File Upload Vulnerabilities | File validation, unrestricted uploads |
| Server-Side Request Forgery (SSRF) | Server-side requests and internal resources |
| XXE Injection | XML external entities and parser weaknesses |
| NoSQL Injection | Injection into NoSQL database queries |
| API Testing | API discovery, endpoint testing, hidden parameters |
| Server-Side Template Injection (SSTI) | Template injection and server-side execution |
| Directory and File Enumeration | Discovering exposed resources and endpoints |
| Race Conditions | Limit overrun and other concurrency-related flaws |

### 2. Advanced Server-Side Topics

- HTTP Host Header Attacks
- HTTP Request Smuggling
- Web Cache Poisoning
- Web Cache Deception
- Insecure Deserialization
- Server-Side Prototype Pollution
- OAuth Authentication Vulnerabilities
- JWT Authentication Vulnerabilities
- WebSockets Security
- GraphQL API Vulnerabilities
- Server-Side Request Forgery (advanced scenarios)

### 3. Client-Side Vulnerabilities

- Cross-Site Scripting (XSS)
- Cross-Site Request Forgery (CSRF)
- Cross-Origin Resource Sharing (CORS)
- Clickjacking
- DOM-Based Vulnerabilities
- Client-Side Prototype Pollution
- Web Message Vulnerabilities
- JavaScript Security Issues

### 4. Authentication and Access Control

- Username Enumeration
- Password Brute-Force Attacks
- Multi-Factor Authentication Bypass
- Password Reset Vulnerabilities
- Session Management
- Broken Access Control
- Horizontal and Vertical Privilege Escalation
- Insecure Direct Object References (IDOR)

### 5. Business Logic and Race Conditions

- Business Logic Flaws
- Workflow Manipulation
- Input Validation Issues
- Trusting Client-Side Controls
- Limit Overrun Race Conditions
- Time-of-Check to Time-of-Use (TOCTOU) Issues
- Exploiting Concurrent Requests

### 6. Web Security Testing Tools

- Burp Suite Proxy
- Burp Suite Repeater
- Burp Suite Intruder
- Burp Suite Scanner
- Burp Suite Decoder
- Burp Suite Comparer
- Burp Suite Collaborator
- Browser Developer Tools
- HTTP Request and Response Analysis

> **Note:** This is a roadmap of relevant learning areas, not a claim that every topic already has completed notes. The repository grows as I study and document each module.

## Repository Structure

The repository is organized by vulnerability category and individual modules wherever practical.

```text
PortSwigger-Web-Security-Academy/
│
├── README.md
│
├── Server-Side/
│   ├── SQL-Injection/
│   ├── Authentication/
│   ├── Access-Control/
│   ├── Path-Traversal/
│   ├── Command-Injection/
│   ├── SSRF/
│   ├── XXE-Injection/
│   ├── API-Testing/
│   ├── Business-Logic/
│   └── Race-Conditions/
│
├── Client-Side/
│   ├── XSS/
│   ├── CSRF/
│   ├── CORS/
│   └── Clickjacking/
│
└── Other-Topics/
    ├── JWT/
    ├── OAuth/
    ├── GraphQL/
    └── WebSockets/
```

*The structure above is an example organizational layout; actual folders may differ based on the repository's current contents.*

## How I Document Each Module

Whenever possible, each module's notes cover:

1. **Overview** — What the vulnerability is.
2. **How It Works** — The underlying technical concept.
3. **Types and Variations** — Important attack scenarios.
4. **Testing Methodology** — How to investigate the vulnerability in authorized labs.
5. **Tools and Techniques** — Relevant Burp Suite features and commands.
6. **Examples** — Representative requests, payloads, or lab observations.
7. **Impact** — Potential security consequences.
8. **Prevention and Mitigation** — How developers can address the issue.
9. **Key Takeaways** — Important points for revision.

Not every module requires every section; the notes follow the content and practical requirements of the topic.

## Learning Resources

- [PortSwigger Web Security Academy](https://portswigger.net/web-security) — Official learning materials and interactive labs.
- [Burp Suite Documentation](https://portswigger.net/burp/documentation) — Documentation for web security testing tools.
- [PortSwigger Research](https://portswigger.net/research) — Security research and technical articles.

## Disclaimer

This repository is intended for **educational purposes, cybersecurity learning, and authorized security testing**.

All practical testing should be performed only in PortSwigger Academy labs, systems you own, or environments where you have explicit permission to test.

The notes are personal study materials and summaries. PortSwigger Web Security Academy and Burp Suite are trademarks of their respective owners. This repository is not an official PortSwigger project.

---

<p align="center">
  <b>Learn • Practice • Document • Improve</b>
  <br/>
  Building practical web application security knowledge, one module at a time.
</p>
