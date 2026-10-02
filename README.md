# NETWORKWALKS-B083-WK4--PENETRATION-TESTING-AND-RISK-ASSESSMENT-REPORT

Engagement Type:	Authorized Black-Box Penetration Test
Target Scope:	Mediroza Hospital Web Application & Patient Portal
Author / Tester:	Afrose Banu S
Date:	2 October 2026

# 01 Assessment Overview

A penetration testing assessment was carried out in an authorized lab environment against the **Mediroza Hospital web application and patient portal**. The objective of the assessment was to identify security weaknesses within the application and determine whether unauthorized access to sensitive patient or organizational information was possible.

The testing identified multiple security issues involving **authentication, authorization, sensitive information exposure, document security, and application configuration**.

A major issue identified during testing was an **SQL Injection vulnerability that resulted in authentication bypass**. This weakness allowed administrative access to the patient portal without the use of legitimate credentials.

Additional testing after gaining access identified sensitive medical PDF documents that were not adequately protected and contained hardcoded password hashes. Analysis of document metadata also exposed information associated with internal backup locations. Testing further demonstrated unauthorized access to confidential hospital administrative information, including **employee salary information and shareholder registers**.

The identified weaknesses create a risk of unauthorized access to sensitive medical and organizational information. Remediation should therefore focus on **secure database input handling, appropriate encryption of sensitive documents, removal of sensitive metadata, and stronger access restrictions for backup resources**.

# 02 Assessment Coverage

## 2.1 Application Under Assessment

The assessment covered the following authorized target:

* **Target Application:** Mediroza Hospital Web Application & Patient Portal
* **Testing Environment:** Authorized Lab Environment

The testing focused on the application's externally accessible functionality, authentication mechanisms, accessible documents, metadata, and resources that could be reached after obtaining application access.

## 2.2 Security Testing Utilities

The following tools and utilities were used during the assessment:

* **Information Gathering:** nslookup, whois, WhatWeb, curl, wafw00f, nmap
* **Application Testing & Data Collection:** Web Browser, curl
* **Document and Metadata Examination:** exiftool, qpdf
* **Hash Analysis & Password Recovery:** Networkwalks Hash Calculator, Networkwalks Password Cracker

# 03 Assessment Approach

The assessment was conducted through a series of testing stages based on **OWASP-aligned penetration testing practices**.

### Phase 1 – Target Discovery and Enumeration

Initial testing focused on gathering information about the target environment. This included identifying application details, reviewing server response headers, checking for the presence of a Web Application Firewall (WAF), and identifying exposed ports and services.

### Phase 2 – Authentication and Authorization Review

The application's authentication and login functionality were examined for weaknesses. Input handling and access control mechanisms were tested to determine whether authentication could be bypassed or restricted functionality could be accessed without valid authorization.

### Phase 3 – Application Access and Data Assessment

Following the identification of access weaknesses, accessible application resources and documents were reviewed. The assessment focused on determining whether sensitive information could be accessed and whether stored documents and password-protected files were adequately secured.

### Phase 4 – Document and Information Exposure Analysis

Accessible files were examined using metadata and document analysis techniques. This stage focused on identifying information that could unintentionally disclose internal system details, file locations, backup paths, or other sensitive information.

The assessment ultimately demonstrated that weaknesses across **authentication, access control, document protection and backup access controls** could expose sensitive hospital information to unauthorized users.

# 01 Findings and Proof of Exploitation

## Step 1: Reconnaissance and Unauthorized Portal Access

### 1.1 Reconnaissance and Service Enumeration

The assessment began with reconnaissance of the target domain/IP to identify the technologies in use, server-related information, DNS details, and exposed services.

The following tools were used during this phase:

* **Nmap:** Used to identify open ports and active web services running on the target host.
* **Wafw00f and WhatWeb:** Used to determine whether a Web Application Firewall (WAF) was present and to identify the technologies used by the web application.
* **Nslookup, Whois, and Curl:** Used to gather DNS information, domain registration details, and HTTP response headers.

The information collected during this phase was used to understand the exposed attack surface and identify areas requiring further security testing.

### 1.2 Patient Portal Authentication Bypass

The authentication mechanism of the Patient Portal was subsequently assessed for input validation and SQL query handling weaknesses.

Testing demonstrated that user-controlled input was not being properly sanitized before being processed by the backend database query.

The following behavior was observed:

* Supplying `admin'` in the username field resulted in a database syntax error, indicating that user input was being incorporated directly into an SQL query.
* Supplying `admin'--` altered the structure of the backend query by commenting out the remaining password validation logic.
* As a result, the application authenticated the user and provided **administrative access without a valid password**.

This demonstrated that the Patient Portal was vulnerable to an **SQL Injection-based authentication bypass**.

### 1.3 Unauthorized Access to Patient Reports

After administrative access was obtained, the patient management functionality was reviewed.

The application allowed access to patient medical reports in PDF format. These reports could be downloaded without the appropriate level of authorization, resulting in exposure of **sensitive patient medical information**.

---

## Step 2: PDF Security Assessment and Password Recovery

### 2.1 Extraction of PDF Password Hashes

The downloaded patient medical reports were protected using PDF document passwords.

To evaluate the strength of the protection mechanism, cryptographic hash values were extracted from the encrypted PDF files using the **Networkwalks Hash Calculator**. The resulting `$pdf$` hash strings were then used for password-strength testing.

### 2.2 Dictionary-Based Password Recovery

The extracted PDF hashes were processed using the **Networkwalks Password Cracker**.

A dictionary-based attack was performed using the available wordlists. The passwords protecting the PDF documents were successfully recovered within seconds, demonstrating that the document passwords were sufficiently weak or predictable to allow practical offline recovery.

This allowed the protected PDF files to be opened using the recovered passwords.

---


### 2.3 Exposure of Database Backup and Confidential Information

Using the backup location identified through the document metadata, an HTTP request was made with **curl** to access the SQL backup file hosted on the web server.

The backup file was accessible directly from the server and could be downloaded without appropriate access restrictions.

Review of the exposed database backup revealed highly sensitive organizational information, including:

* Staff salary details
* Shareholder information
* Internal system credentials

This demonstrated that information disclosed through document metadata could be used to identify an exposed internal resource containing **highly confidential corporate and administrative data**.

---

# 03 Risk Rating & Vulnerability Assessment

The following vulnerabilities were identified during the assessment. Severity levels and CVSS v3.1 scores are based on the potential impact and exploitability demonstrated during testing.

| Vulnerability                             | Severity     | CVSS v3.1 | Impact Summary                                                                                        |
| ----------------------------------------- | ------------ | --------: | ----------------------------------------------------------------------------------------------------- |
| **SQL Injection (Authentication Bypass)** | **Critical** |   **9.8** | Allows unauthorized administrative access to the portal and exposure of patient records.              |
| **Exposed Database Backup File**          | **Critical** |   **9.1** | Allows direct access to confidential corporate information, financial records, and staff salary data. |
| **Weak PDF Encryption Passwords**         | **High**     |   **7.5** | Enables offline password recovery and unauthorized access to protected patient documents.                  |

---

# 04 Recommendations and Remediation

## 1. Address SQL Injection and Authentication Bypass — Critical

### Parameterized Queries and Prepared Statements

All database operations should use **parameterized queries or prepared statements**. User-supplied values must never be directly concatenated into SQL statements.

Technologies such as **PDO prepared statements in PHP** or **PreparedStatement in Java** should be used where applicable.

### Input Validation and Sanitization

Implement strict input validation across all application forms and parameters. Where possible, use **allowlist-based validation** to ensure that only expected input formats and values are accepted.

---

## 2. Protect Database Backups and Restrict Direct File Access — Critical

### Store Backups Outside the Web Root

Database backups and internal configuration files should not be stored within publicly accessible web directories.

Sensitive files should be stored outside the web server's document root, such as outside `/var/www/html/`, with access controlled through appropriate application or administrative mechanisms.

### Apply File and Directory Access Controls

Web server access rules should prevent direct public access to sensitive file types and backup resources.

Restrictions should be applied to files such as:

* `.sql`
* `.bak`
* `.log`

Appropriate **Apache `.htaccess` rules or Nginx location-based access controls** should be implemented where applicable.

---

## 5. Strengthen PDF Encryption and Password Policies — High

### Enforce Strong Password Requirements

Passwords used to protect PDF documents and user accounts should follow strong password requirements.

A minimum length of **12 characters or more** should be enforced, with a combination of uppercase and lowercase letters, numbers, and special characters.

Passwords should also avoid predictable words, patterns, or information that could be easily associated with the organization or its users.

### Implement Multi-Factor Authentication

**Multi-Factor Authentication (MFA)** should be enabled for administrative access to the Patient Portal.

MFA provides an additional authentication layer and can help reduce the impact of compromised or bypassed credentials.

## 6. Evidence and Proof of exploitation

<img width="1912" height="855" alt="Reconnaisance Pentesting" src="https://github.com/user-attachments/assets/8bdee536-42b3-476d-aa10-641472d6582e" />
<img width="1917" height="970" alt="Pentest " src="https://github.com/user-attachments/assets/18c357a5-2066-4eb7-9fc0-90c0c18e7a5d" />
<img width="752" height="345" alt="WAF - " src="https://github.com/user-attachments/assets/52054245-6be2-4801-9ccc-730ee59ac050" />
<img width="640" height="587" alt="SQL" src="https://github.com/user-attachments/assets/ed02a71f-8454-49fb-8019-0417b8b1f35c" />

<img width="742" height="105" alt="exfiltration of sensitive patient reports" src="https://github.com/user-attachments/assets/498ef71c-9982-4035-8c33-951b497e4bb6" />
<img width="1055" height="517" alt="Hash calculator network walks " src="https://github.com/user-attachments/assets/d4a39442-dc47-4966-9955-9c798998ce5b" />
<img width="1510" height="622" alt="Lab report of  3 patients - Pentesting" src="https://github.com/user-attachments/assets/1c1e2c04-fe06-4949-afe4-a622fa960058" />

<img width="1487" height="646" alt="loggedin - sql injection pentetsing" src="https://github.com/user-attachments/assets/5622b9e3-b9f7-49aa-871d-420a75e806eb" />
<img width="1262" height="792" alt="Password cracking - network walks" src="https://github.com/user-attachments/assets/9dc16034-0748-4a5a-a39d-9b9fc57520fa" />
<img width="955" height="641" alt="Password cracking - pdf 1 " src="https://github.com/user-attachments/assets/f2a28add-bd53-4d77-8f6e-ad67c0dcee12" />
<img width="725" height="287" alt="Patient data " src="https://github.com/user-attachments/assets/5b80b393-5adf-4616-8e23-34c78099402d" />
<img width="756" height="762" alt="Patient reports" src="https://github.com/user-attachments/assets/fe2e76f9-0cbc-426e-9ae3-cf853f7f563a" />
<img width="1005" height="646" alt="pdf 2 - opened with password ( pentesting)" src="https://github.com/user-attachments/assets/7dd8469a-5a2a-478d-b7d1-f32428f11cd6" />
<img width="927" height="637" alt="PDF 2 - Password cracking" src="https://github.com/user-attachments/assets/1c57f8cd-af42-4e68-8fe4-b92932e14051" />

<img width="942" height="447" alt="PDF 3 - Password cracking " src="https://github.com/user-attachments/assets/a68821d8-bfb8-4426-aa32-bcc3c2e2a7dd" />
<img width="945" height="611" alt="pdf 3 - password cracking" src="https://github.com/user-attachments/assets/5ff03c99-4b69-460d-b3dc-517889264db3" />




# Disclaimer

This penetration testing report has been prepared **strictly for educational and learning purposes**. All security testing, vulnerability assessment, exploitation, and data analysis activities described in this report were conducted within an **authorized website, lab environment, and testing scope**.

No unauthorized systems, websites, networks, or applications were targeted during the assessment. The techniques and tools documented in this report were used solely to understand and demonstrate cybersecurity vulnerabilities in the authorized environment.

The findings and evidence presented in this report are intended for **educational, training, and cybersecurity skill-development purposes only** and should not be used to perform unauthorized testing or access systems without explicit permission.




