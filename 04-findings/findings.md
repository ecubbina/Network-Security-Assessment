# Security Findings

## Network Security Assessment — Metasploitable 2

This document summarizes the security findings identified during an authorized network security assessment of the Metasploitable 2 virtual machine.

> **Assessment Type:** Authorized vulnerability assessment / security testing  
> **Target:** Metasploitable 2  
> **Target IP:** 192.168.27.129  
> **Assessment Platform:** Kali Linux  
> **Network:** Isolated VMware Host-Only Network  
> **Date:** 2026

---

## Risk Rating Summary

| ID | Finding | Severity | Status |
|---|---|---|---|
| F-01 | Unauthenticated WebDAV File Creation and Deletion | High | Confirmed |
| F-02 | Broad NFS Export of Root Filesystem | High / Medium-High | Confirmed |
| F-03 | Anonymous SMB Access to `tmp` Share | Medium | Confirmed |
| F-04 | Anonymous FTP and Cleartext Authentication | Medium | Confirmed |
| F-05 | SMB Signing Not Required | Medium | Confirmed |
| F-06 | SMTP SSLv2 and Weak Cryptographic Support | Medium | Confirmed |
| F-07 | Application Information Disclosure | Low-Medium | Confirmed |
| F-08 | Legacy Software and Excessive Attack Surface | Low-Medium | Observation |

---

# F-01 — Unauthenticated WebDAV File Creation and Deletion

**Severity:** High  
**Status:** Confirmed

### Description

The Apache WebDAV endpoint at `/dav/` permits unauthenticated users to create and delete files.

The WebDAV service advertised methods including:

- PUT
- DELETE
- PROPFIND
- PROPPATCH
- COPY
- MOVE
- LOCK
- UNLOCK

Testing confirmed that a file could be created without authentication using the HTTP `PUT` method.

### Evidence

A test file was successfully created:

```text
PUT /dav/project1-test.txt
HTTP/1.1 201 Created


The file was subsequently retrieved successfully:

GET /dav/project1-test.txt
HTTP/1.1 200 OK

The file was then deleted:

DELETE /dav/project1-test.txt
HTTP/1.1 204 No Content

A subsequent request returned:

HTTP/1.1 404 Not Found


Impact

An unauthenticated attacker could potentially:

Create unauthorized files
Modify application content
Delete accessible files
Abuse the WebDAV service as an attack surface
Potentially facilitate further attacks depending on server configuration

Remote code execution was not claimed or demonstrated as part of this assessment.

Recommendation
Disable WebDAV if it is not required.
Require authentication and authorization for WebDAV resources.
Restrict allowed HTTP methods.
Apply least-privilege permissions to the WebDAV directory.
Monitor WebDAV activity and file modifications.
Separate writable directories from executable web content.
Evidence Location

See:

03-Wireshark/

and the HTTP testing evidence captured during the assessment.


F-02 — Broad NFS Export of Root Filesystem

Severity: High / Medium-High
Status: Confirmed

Description

The target exposed an NFS export for the root filesystem:

/

The export was available to:

*

This indicates that the root filesystem was broadly exported to any host able to reach the NFS service.

Evidence

The following command identified the export:

showmount -e 192.168.27.129

Result:

Export list for 192.168.27.129:
/ *

Nmap enumeration also identified NFS services on the target.

Impact

Broad NFS exports can expose sensitive filesystem resources to unauthorized systems.

Depending on export options and filesystem permissions, an attacker may potentially:

Access sensitive files
Read configuration information
Access credentials or application data
Modify files where write access is permitted
Abuse improperly configured filesystem permissions

This assessment did not claim root access solely from the export configuration.

Recommendation
Do not export the root filesystem.
Restrict NFS exports to specific trusted hosts or networks.
Use read-only exports where possible.
Configure appropriate root squashing.
Apply least-privilege filesystem permissions.
Disable NFS services if they are not required.


F-03 — Anonymous SMB Access to tmp Share

Severity: Medium
Status: Confirmed

Description

The SMB service allowed anonymous authentication and provided access to the tmp share without requiring credentials.

Evidence

Anonymous SMB enumeration was performed using:

smbclient -L //192.168.27.129 -N

The target responded:

Anonymous login successful

The following shares were identified:

print$
tmp
opt
IPC$
ADMIN$

Anonymous access to the tmp share was confirmed:

smbclient //192.168.27.129/tmp -N

The connection was successful and directory contents were visible.

Impact

Anonymous SMB access can expose files and system resources to unauthenticated users.

Potential risks include:

Unauthorized information disclosure
Exposure of temporary files
Increased attack surface
Potential abuse if write permissions are available

Write access was not claimed or demonstrated during this assessment.

Recommendation
Disable guest/anonymous SMB access where unnecessary.
Restrict share permissions using least privilege.
Remove unnecessary SMB shares.
Require authentication for sensitive resources.
Monitor anonymous SMB activity.


F-04 — Anonymous FTP and Cleartext Authentication

Severity: Medium
Status: Confirmed

Description

The FTP service permitted anonymous authentication.

FTP also transmits authentication information without encryption when TLS is not used.

Evidence

Anonymous login was successfully performed:

USER anonymous
331 Please specify the password.

PASS anonymous
230 Login successful.

Network traffic captured using Wireshark confirmed that FTP commands were transmitted in cleartext.

Impact

An attacker with the ability to observe network traffic could potentially capture FTP credentials and session information.

Anonymous access can also expose files or resources that should not be publicly accessible.

Recommendation
Disable anonymous FTP unless explicitly required.
Replace FTP with SFTP or another encrypted file-transfer protocol.
If FTP must be retained, enforce TLS.
Restrict FTP access using network controls.
Monitor authentication activity.
Evidence Location

Wireshark traffic analysis is documented in:

03-Wireshark/


F-05 — SMB Signing Not Required

Severity: Medium
Status: Confirmed

Description

SMB enumeration identified that message signing was disabled or not required.

Evidence

Nmap SMB enumeration reported that SMB message signing was:

disabled / not required
Impact

When SMB signing is not required, an attacker may have greater opportunity to perform certain man-in-the-middle or relay-style attacks in environments where other conditions are also present.

This assessment did not perform an SMB relay attack.

Recommendation
Require SMB message signing.
Apply secure SMB configuration policies.
Disable outdated SMB protocols where possible.
Monitor unusual SMB authentication and network activity.


F-06 — SMTP SSLv2 and Weak Cryptographic Support

Severity: Medium
Status: Confirmed

Description

The SMTP service supported legacy SSLv2 functionality and weak cryptographic options.

Evidence

Nmap service enumeration identified:

SMTP
Postfix
SSLv2 supported
Weak SSLv2 ciphers
Impact

Legacy cryptographic protocols and weak cipher support can reduce the confidentiality and integrity of encrypted communications.

Recommendation
Disable SSLv2.
Disable obsolete cryptographic protocols.
Require modern TLS configurations.
Use strong cipher suites.
Regularly review mail-server TLS configuration.


F-07 — Application Information Disclosure

Severity: Low-Medium
Status: Confirmed

Description

The Mutillidae web application exposed internal application information through client-visible HTML source code.

The application source contained comments referencing database configuration information.

The page also disclosed application and environment information including:

Version: 2.1.19
PHP Version: 5.2.4-2ubuntu5.10
Impact

Information disclosure can provide attackers with useful intelligence about:

Application versions
Backend technologies
Configuration
Potential attack paths

The assessment did not assume that the database password mentioned in the HTML comment was valid.

Recommendation
Remove sensitive comments from production HTML.
Avoid exposing configuration information to clients.
Remove unnecessary version information.
Keep applications and dependencies updated.
Review source code before deployment.


F-08 — Legacy Software and Excessive Attack Surface

Severity: Low-Medium
Status: Observation

Description

The target exposed numerous network services and legacy software versions.

Examples identified during enumeration included:

vsftpd 2.3.4
OpenSSH 4.7p1
Apache 2.2.8
Samba 3.0.20
MySQL 5.0.51a
PostgreSQL 8.3
Apache Tomcat 5.5
UnrealIRCd 3.2.8.1
Impact

Legacy software may contain known vulnerabilities and may no longer receive security updates.

Exposing a large number of unnecessary services also increases the overall attack surface.

Recommendation
Remove unnecessary services.
Upgrade unsupported software.
Apply security patches regularly.
Restrict management interfaces.
Use firewall rules to limit service exposure.
Follow the principle of least functionality.

Note: The presence of an old software version alone was not treated as proof of a specific exploitable vulnerability.


Security Observations

The following services were identified during the assessment but were not classified as confirmed vulnerabilities based solely on the observed behavior.

Tomcat Manager

The Tomcat Manager interface returned:

401 Unauthorized

Authentication was therefore required.

Tomcat Status

The Tomcat status interface also returned:

401 Unauthorized
Tomcat Administration

The /admin/ interface presented an authentication page rather than providing unauthenticated administrative access.

phpMyAdmin

The phpMyAdmin interface was accessible but required authentication.

DVWA

The Damn Vulnerable Web Application login interface was accessible but authentication was required.

These observations demonstrate that service exposure does not automatically mean unauthorized access was achieved.


Security Observations

The following services were identified during the assessment but were not classified as confirmed vulnerabilities based solely on the observed behavior.

Tomcat Manager

The Tomcat Manager interface returned:

401 Unauthorized

Authentication was therefore required.

Tomcat Status

The Tomcat status interface also returned:

401 Unauthorized
Tomcat Administration

The /admin/ interface presented an authentication page rather than providing unauthenticated administrative access.

phpMyAdmin

The phpMyAdmin interface was accessible but required authentication.

DVWA

The Damn Vulnerable Web Application login interface was accessible but authentication was required.

These observations demonstrate that service exposure does not automatically mean unauthorized access was achieved.


Remediation Priority
Priority 1 — Immediate
Disable or secure unauthenticated WebDAV access.
Remove the broad NFS root filesystem export.
Restrict anonymous SMB access.
Disable anonymous FTP.
Priority 2 — High
Require SMB signing.
Disable SSLv2 and weak cryptographic protocols.
Restrict unnecessary network services.
Priority 3 — Medium
Remove information disclosure from web applications.
Upgrade unsupported software.
Review exposed services and firewall rules.


Conclusion

The assessment successfully demonstrated a structured network security assessment methodology involving:

Network discovery
Port scanning
Service enumeration
Protocol analysis
SMB/NFS enumeration
Web service testing
WebDAV access validation
Wireshark traffic analysis
Risk assessment
Security remediation planning

All testing was performed against an intentionally vulnerable Metasploitable 2 virtual machine within an isolated lab environment.

No production systems or unauthorized external systems were targeted.
