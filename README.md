# Tryhackme-rabbit-store-writeup
Red Team penetration testing report demonstrating a full kill chain from web access to root compromise on the TryHackMe Rabbit Store machine.

🐇 Rabbit Store Web Application
Penetration Testing Report & Kill Chain Walkthrough
1. Introduction & Scope
1.1 Purpose of the Report

This report documents the results of a penetration test conducted against the Rabbit Store web application.
The primary objective of this assessment was to proactively evaluate the security posture of the application, identify exploitable vulnerabilities, and demonstrate their real-world impact by chaining them together into a full system compromise.

The test was performed from an attacker’s perspective and focused on simulating realistic attack scenarios rather than isolated vulnerability checks.

The main goals of this report are:

To clearly define the scope of the Rabbit Store application.

To document the methodology and tools used during testing.

To provide step-by-step exploitation details so that learners can reproduce the attack.

To chronologically document the kill chain, from initial reconnaissance to full root compromise.

2. Methodology & Tools
2.1 Testing Methodology

A structured external-to-internal kill chain methodology was used.
Each phase built upon the information gathered in the previous stage, allowing vulnerabilities to be exploited in context rather than isolation.

Testing Phases:

Reconnaissance – Identifying exposed services, ports, and subdomains.

Initial Access – Abusing a web logic flaw to gain privileged application access.

Enumeration – Discovering internal services and undocumented API endpoints.

Exploitation – Achieving remote command execution via SSTI.

Privilege Escalation – Pivoting through RabbitMQ and misconfigurations to obtain root access.

2.2 Tools Used
Tool	Purpose
Nmap	Port scanning and service identification (SSH, HTTP, Erlang, RabbitMQ)
FFuf	Subdomain and API endpoint enumeration
Burp Suite	Intercepting and manipulating HTTP requests (JWT abuse)
pspy	Monitoring background processes for privilege escalation
Custom Python Scripts	SSRF enumeration and Erlang exploitation
revshells.com	Generating reverse shell payloads
rabbitmqctl / rabbitmqadmin	RabbitMQ user management and configuration export
3. Findings & Exploitation Chain

This section explains how individual vulnerabilities were chained together to achieve full system compromise.

3.1 Phase 1 – Reconnaissance & Enumeration

Nmap scanning revealed the following open ports:

22   SSH
80   HTTP
4369 Erlang Port Mapper (epmd)
25672 RabbitMQ


FFuf subdomain enumeration identified:

storage.cloudsite.thm


The storage subdomain hosted a functional web application with authentication features, significantly expanding the attack surface.

3.2 Phase 2 – Privileged Web Access (JWT Logic Flaw)

A user registration request was intercepted using Burp Suite.

The request body was a JSON object containing email and password.

By manually adding the parameter:

"subscription": "active"


the server issued a privileged JWT without validation.

Impact:

Administrative approval was bypassed.

Restricted application features (file upload, internal fetch functionality) became accessible.

3.3 Phase 3 – Internal Network Discovery (SSRF)

The “Upload from URL” feature caused the server to fetch user-supplied URLs.

This behavior enabled Server-Side Request Forgery (SSRF).

Requests to:

http://127.0.0.1:<port>


were successfully processed.

Using a custom Python script, an internal service was discovered on:

Port 3000


Accessing:

http://127.0.0.1:3000/api/docs


exposed undocumented API endpoints.

3.4 Phase 4 – Remote Code Execution (SSTI)

The endpoint:

/api/fetch_messages_from_chatbot


reflected user-controlled input in the response.

Injecting Jinja2 payloads confirmed a Server-Side Template Injection (SSTI) vulnerability.

The payload successfully executed system commands, proving Remote Code Execution (RCE).

3.5 Phase 5 – Foothold (azrael User)

A base64-encoded reverse shell payload was generated.

The payload was executed via SSTI.

An interactive shell was obtained as the azrael user.

3.6 Phase 6 – Lateral Movement (rabbitmq User)

pspy revealed frequent RabbitMQ-related background processes.

Investigation of /var/lib/rabbitmq/ exposed the .erlang.cookie file.

Using this cookie, a custom Erlang exploit (shell-erldp2.py) was executed.

A second shell was obtained as the rabbitmq user.

3.7 Phase 7 – Final Privilege Escalation (root)

A critical clue was found inside RabbitMQ database files:

rabbit_user.DCD


indicating that the system root password matched the RabbitMQ root SHA-256 hash.

Steps performed:

Created a new RabbitMQ administrator user (0xb0b).

Exported RabbitMQ definitions:

rabbit.definitions.json


Extracted the base64-encoded root password hash.

Decoded the hash to obtain the raw SHA-256 value.

Used the hash itself as the root password:

su root


Root access was successfully obtained.

4. Conclusion

This assessment demonstrates that the Rabbit Store web application suffers from multiple critical security weaknesses across both application and infrastructure layers.

A seemingly minor JWT logic flaw acted as the initial entry point, ultimately enabling:

Internal network access via SSRF

Remote code execution via SSTI

Lateral movement through RabbitMQ

Full root compromise due to unsafe credential handling

The findings highlight how chained vulnerabilities, even if individually moderate, can result in complete system takeover when combined.

⚠️ Disclaimer

This report was created solely for educational purposes within a controlled TryHackMe environment.
No real-world systems were targeted.

🔗 Tags

#TryHackMe #PenetrationTesting #RedTeam #KillChain #SSRF #SSTI #RabbitMQ #Erlang #PrivilegeEscalation
