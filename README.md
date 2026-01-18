# Tryhackme-rabbit-store-writeup
Red Team penetration testing report demonstrating a full kill chain from web access to root compromise on the TryHackMe Rabbit Store machine.

# Rabbit Store Web Application

## Penetration Testing Report (Red Team Case Study)

## Executive Summary

This report presents the results of a penetration test conducted against the **Rabbit Store web application**, hosted within a TryHackMe laboratory environment.
The assessment demonstrates how multiple security weaknesses across the **web application, internal services, and message queue infrastructure** can be chained together to achieve **full system compromise (root access)**.

The engagement followed a **kill chain–driven methodology**, simulating realistic attacker behavior from initial reconnaissance through privilege escalation.

> **Impact:**
> An unauthenticated remote attacker can fully compromise the underlying system, gaining root-level access.

---

## Scope & Objectives

### Scope

* Rabbit Store Web Application
* Associated internal services
* RabbitMQ & Erlang-based infrastructure

### Objectives

* Identify exploitable vulnerabilities
* Demonstrate real-world attack paths
* Document a complete attack chain
* Provide reproducible, educational findings

---

## Methodology

The test followed a structured **external-to-internal penetration testing methodology**, where each phase informed the next.

**Assessment Phases:**

1. Reconnaissance
2. Initial Access
3. Internal Enumeration
4. Exploitation
5. Lateral Movement
6. Privilege Escalation

This approach ensured vulnerabilities were evaluated **in context**, rather than isolation.

---

## Tools Used

| Tool                        | Purpose                                     |
| --------------------------- | ------------------------------------------- |
| Nmap                        | Network scanning and service discovery      |
| FFuf                        | Subdomain and endpoint enumeration          |
| Burp Suite                  | HTTP interception and request manipulation  |
| pspy                        | Process monitoring for privilege escalation |
| Custom Python Scripts       | SSRF enumeration & Erlang exploitation      |
| revshells.com               | Reverse shell payload generation            |
| rabbitmqctl / rabbitmqadmin | RabbitMQ administration & data export       |

---

## Attack Chain Overview

### Phase 1 – Reconnaissance

* Identified exposed services: SSH, HTTP, Erlang Port Mapper, RabbitMQ
* Discovered `storage.cloudsite.thm` subdomain hosting the application

---

### Phase 2 – Application Logic Flaw (JWT Manipulation)

* User registration requests were intercepted
* Adding an unvalidated JSON parameter:

  ```json
  "subscription": "active"
  ```

  resulted in issuance of a **privileged JWT**
* Administrative approval was bypassed

**Impact:** Unauthorized access to restricted application features

---

### Phase 3 – Internal Network Access (SSRF)

* “Upload from URL” feature enabled server-side HTTP requests
* SSRF was exploited to access:

  ```
  http://127.0.0.1:3000
  ```
* Internal API documentation was exposed via `/api/docs`

---

### Phase 4 – Remote Code Execution (SSTI)

* The internal API reflected user-controlled input
* A **Server-Side Template Injection (SSTI)** vulnerability was confirmed
* Arbitrary command execution was achieved on the server

---

### Phase 5 – Initial Foothold

* SSTI was leveraged to execute a reverse shell
* Interactive shell access obtained as user **azrael**

---

### Phase 6 – Lateral Movement (RabbitMQ)

* `pspy` revealed RabbitMQ-related background processes
* `.erlang.cookie` was recovered from `/var/lib/rabbitmq/`
* Erlang Distribution Protocol was abused
* Shell access obtained as **rabbitmq** user

---

### Phase 7 – Privilege Escalation (Root)

* RabbitMQ database files revealed a critical configuration flaw
* The system root password matched the RabbitMQ root SHA-256 hash
* RabbitMQ definitions were exported
* The base64-encoded hash was decoded
* The raw SHA-256 hash was accepted as the root password

**Result:** Full root access obtained

---

## Impact Assessment

* Complete system compromise
* Full administrative control
* Potential for data exfiltration
* Infrastructure pivoting risk
* Service disruption capability

---

## Key Takeaways

* Internal services must never be implicitly trusted
* Message queue systems represent high-value attack surfaces
* Logic flaws can act as critical entry points
* Chained vulnerabilities significantly amplify risk
* Credential handling failures directly lead to full compromise

---

## Conclusion

The Rabbit Store application contains multiple critical security weaknesses across both application and infrastructure layers.
When combined, these weaknesses enable an attacker to progress from unauthenticated access to full root compromise.

This assessment highlights the importance of **defense-in-depth**, strict validation of trust boundaries, and secure credential management.

---

## Disclaimer

This report was created strictly for **educational purposes** within a controlled TryHackMe environment.
No real-world systems were targeted.

---

## Tags

`#RedTeam` `#PenetrationTesting` `#KillChain` `#TryHackMe` `#RabbitMQ` `#Erlang` `#SSRF` `#SSTI` `#PrivilegeEscalation`
