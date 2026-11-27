
<div align="center">

# 🔐 Cybersecurity Week 4: Penetration Testing Report: Mediroza General Hospital

**Application Security Assessment: Penetration Testing Report**

</div>
<p align="center">
  <img src="https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000" /> <img src="https://img.shields.io/badge/Skill-Linux-404040?style=flat-square&labelColor=C00000" /><img src="https://img.shields.io/badge/Skill-Virtualization-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Ver-Virtualbox%20v7.2-0070C0?style=flat-square&labelColor=000000" /><img src="https://img.shields.io/badge/Kali%20Linux-v2026.2-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Penetration%20Testing-C00000?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/Ethical%20Hacking-E87500?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/GitHub-404040?style=flat-square&labelColor=0070C0&logo=github&logoColor=white" />
  <img src="https://img.shields.io/badge/Kali%20Linux-404040?style=flat-square&labelColor=C00000&logo=kalilinux&logoColor=white" />
  <img src="https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000" />
  <img src="https://img.shields.io/badge/Waqas%20Karim%20CCIE-C00000?style=flat-square" />
</p>


## 📂 Table of Contents

* [⚠️ Disclaimer & Authorization](#disclaimer)
* [✅ Completed Milestones Summary](#milestones-summary)
* [01. 📌 Executive Summary](#executive-summary)
* [02. 🎯 Penetration Scope and Methodology](#scope-and-methodology)
  * [🚧 Project Boundaries](#project-boundaries)
  * [🏗️ Lab Architecture](#lab-architecture)
  * [🔬 Testing Approach](#testing-approach)
  * [🧱 Limitations & Constraints Encountered](#-limitations--constraints-encountered)
* [03. 🔍 Findings and Proof of Exploitation](#findings-and-exploitation)
  * [Finding 1: Error-Based SQL Injection (Milestone 1)](#finding-1)
  * [Finding 2: Insecure Cryptographic Storage & Low-Entropy PDF Passwords (Milestone 2)](#finding-2)
  * [Finding 3: Unauthenticated Database Backup Exposure (Milestone 3)](#finding-3)
* [04. ⚖️ Risk Rating Matrix](#risk-rating)
* [05. 🛡️ Recommendations and Remediation](#recommendations-remediation)
* [06. Tools and Resources](#tools-and-resources)


---

<a id="disclaimer"></a>
## ⚠️ Disclaimer & Authorization 
All penetration testing activities were performed in a controlled educational lab environment against authorized target infrastructure `https://medirozahospital.com`, simulating Mediroza General Hospital. The target was explicitly authorized for security testing by Networkwalks as part of the Networkwalks Cybersecurity Program. These techniques must never be applied to any system without explicit written permission from the owner.

---

<a id="milestones-summary"></a>
## ✅ Completed Milestones Summary

* 🌐 **Milestone 1 (Initial Access & Reconnaissance):** Conducted passive OSINT, discovered `robots.txt` metadata leaks (`/old/`), identified an unparameterized error-based SQL Injection vulnerability on `/patient/login.php`, bypassed authentication, and retrieved 3 encrypted patient PDF reports.
* 🕵️‍♂️ **Milestone 2 (Data Extraction & Decryption):** Extracted cryptographic hashes for the first two patient reports (`patient_report_1.pdf` and `patient_report_2.pdf`) using the Networkwalks Hash Calculator and recovered their plaintext passwords via the Networkwalks Password Cracker using its default built-in wordlist. For the third report (`patient_report_3.pdf`), extracted the cryptographic hash using `pdf2john` and recovered the plaintext PDF encryption password using John the Ripper (`john`) and the Networkwalks Password Cracker tool with the `rockyou.txt` wordlist.
* 💥 **Milestone 3 (Critical Data Exposure):** Audited an unauthenticated legacy directory (`/old/`) containing `mediroza_db_backup_2019.sql`, exposing 30 internal employee salary records (up to R160,000/mo) and 100% of corporate shareholder equity distributions.
* 📑 **Milestone 4 (Penetration Testing Documentation):** Compiled all technical findings, risk ratings, evidence, troubleshooting logs, and remediation code into this unified pentest documentation repository.

---

<a id="executive-summary"></a>
## 01. 📌 Executive Summary

This report focuses on Mediroza General Hospital who partnered with Networkwalks to conduct a 5-day black-box penetration test and web application security assessment of its primary public web infrastructure (`https://medirozahospital.com`). The primary objective of this engagement was to evaluate the application's overall security posture, test authentication controls on patient-facing portals, identify unintended data exposure vectors, and assess backend storage safeguards protecting sensitive electronic health records (EHR) and corporate assets.

### 🎯 Key Assessment Discoveries
1. **Authentication Bypass & Unrestricted EHR Access (Milestone 1):** An error-based SQL Injection vulnerability on the Patient Portal login endpoint (`/patient/login.php`) allowed an unauthenticated remote attacker to inject SQL logic (`admin' -- `), bypass authentication without valid credentials, and gain full access to private patient pathology lab reports.

2. **Weak PDF Cryptographic Protections (Milestone 2):** Downloaded patient pathology reports were encrypted using low-entropy password policies. Utilizing the **Networkwalks Hash Calculator** and **Networkwalks Password Cracker** (with default wordlist) for `patient_report_1.pdf` and `patient_report_2.pdf`, and Kali Linux utilities (**`pdf2john`** and **John the Ripper** / **Networkwalks Cracker** with `rockyou.txt`) for `patient_report_3.pdf`, all encryption keys were successfully cracked to reveal plaintext patient lab data.
3. **Unauthenticated Financial & HR Database Exposure (Milestone 3):** Inspection of metadata in `robots.txt` disclosed an unlinked directory (`/old/`) with directory listing enabled. This exposed an unencrypted legacy database dump (`mediroza_db_backup_2019.sql`), leaking full South African National ID numbers, contact details, and monthly payroll data for 30 staff members (ranging up to R160,000/mo), alongside 100% of the hospital's corporate shareholder equity structure across 10 shareholders.

### ⚖️ Engagement Risk Assessment
* **Overall Engagement Risk Rating:** `🔴 CRITICAL`
* **Primary Threat Category:** System-Wide Data Breach & Confidentiality Compromise
* **Regulatory & Legal Impact:** Severe non-compliance with the South African **Protection of Personal Information Act (POPIA)** and international **HIPAA** standards due to unauthenticated exposure of Personally Identifiable Information (PII) and confidential health data.

---

<a id="scope-and-methodology"></a>
## 02. 🎯 Penetration Scope and Methodology 

This assessment focuses on evaluating the security of Mediroza General Hospital’s external web systems from an attacker’s perspective. The goal is to identify application vulnerabilities, test how user inputs are validated, review session management controls, and determine whether confidential healthcare data is properly protected.

---

<a id="project-boundaries"></a>
### 🚧 Project Boundaries

| Field | Details |
| :--- | :--- |
| **Target Organization** | Mediroza General Hospital (14 Cedar Avenue, Johannesburg, South Africa) |
| **Client Facility Overview** | 250-Bed Accredited General Hospital (40+ Medical Specialists) |
| **Target URL** | `https://medirozahospital.com` |
| **Resolved Target IP** | `199.188.201.16` |
| **Hosting Infrastructure** | Namecheap, Inc. Shared Web Hosting (`AS22612` / `NCNET-1`) |
| **Web Server Stack** | LiteSpeed Web Server • PHP v8.2.36 • MySQL Database Engine |
| **In-Scope Endpoints** | Primary Web Application (`/`, `/patient/login.php`, `/staff/login.php`, `/old/`) |
| **Security Tester** | **Merin Jayeson** — *Cybersecurity Professional (Batch B084)* |
| **Sponsoring Body** | Networkwalks Cybersecurity Program |
| **Authorization Status** | Explicit Written Authorization Granted |
| **Out-of-Scope Limits** | No Denial of Service (DoS/DDoS), no physical security testing, no social engineering |
| **Technical Testing Standards** | OWASP Web Security Testing Guide (`WSTG v4.2`)|
| **Regulatory Frameworks** | South African **POPIA** (Protection of Personal Information Act) |
| **Assessment Duration** | 5 Days |


--- 

<a id="lab-architecture"></a>
### 🏗️ Lab Architecture

Testing was performed using a dual-OS virtualization environment, isolating passive reconnaissance and web proxy interception within an authorized Kali Linux virtual machine.

```text
[ Windows 11 Host PC ] ──► [ VirtualBox v7.2 ] ──► [ Kali Linux VM (2026.3) ]
                                                        ├── Firefox Browser + DevTools (F12)
                                                        ├── Burp Suite Proxy (127.0.0.1:8080)
                                                        ├── Passive OSINT (theHarvester, dig, whois)
                                                        ├── Online Tools (Networkwalks Hash Calculator & Cracker)
                                                        └── CLI Tools (pdf2john & John the Ripper)



```
---

<a id="testing-approach"></a>
### 🔬 Testing Approach

The assessment followed the 4-stage penetration testing methodology defined by **OWASP WSTG v4.2**:


1. **Phase 1: Passive Reconnaissance & OSINT (OWASP WSTG-INFO-01)**
   * Executed zero-contact intelligence gathering using `theHarvester` (`theHarvester -d medirozahospital.com -l 50 -b all`), `dig`, and `whois`.
   * Mapped target IP `199.188.201.16` to Namecheap shared hosting (`AS22612` / `NCNET-1`) and enumerated subdomains (`ftp.medirozahospital.com`, `heartpatient.medirozahospital.com`, `mail.medirozahospital.com`) via DNS fallback queries.
   * Confirmed the absence of third-party DNS-level WAFs or CDNs (such as Cloudflare or AWS WAF).

2. **Phase 2: Web Application Enumeration & Proxy Transition (OWASP WSTG-INFO-02 to WSTG-INFO-05)**
   * Inspected HTTP response headers (`Server: LiteSpeed`) and metadata files (`robots.txt`), discovering `Disallow: /patient/`, `Disallow: /staff/`, and an unlinked legacy directory `Disallow: /old/`.
   * Initial form mapping and header inspection were performed in Firefox Developer Tools (F12 Network tab). Due to DevTools limitations in intercepting in-flight requests and modifying raw HTTP `POST` body payloads dynamically, testing pivoted to **Burp Suite Community Edition**.
   * Configured Firefox manual proxy settings (`127.0.0.1:8080`), verified active proxy listener status, and validated local CA certificate trust for HTTPS inspection on `https://medirozahospital.com`.

3. **Phase 3: Active Vulnerability Assessment & Exploitation (OWASP WSTG-INPV-05 & WSTG-ERRH-01)**
   * Navigated to `/old/` directory listing and retrieved an unauthenticated MySQL database dump (`mediroza_db_backup_2019.sql`), exfiltrating 30 staff salary records and 10 corporate shareholder equity distributions.
   * Conducted single-character boundary checks on `/patient/login.php` by submitting a single quote (`'`), observing explicit `"Username not found"` enumeration vs. raw MySQL syntax errors (`Warning: mysqli_query()`).
   * Captured the login request in Burp Suite Proxy, forwarded it to Burp Repeater, and injected the SQL authentication bypass payload (`admin' -- `), receiving an HTTP `302 Found` redirect to `portal.php` and obtaining access to 3 encrypted patient PDF pathology reports.

4. **Phase 4: Cryptographic Analysis & Remediation Reporting (OWASP WSTG-CRYP-01)**
   * Extracted `$pdf$` encryption hashes using the **Networkwalks Hash Calculator** (`patient_report_1.pdf` and `patient_report_2.pdf`) and Kali's **`pdf2john`** (`patient_report_3.pdf`).
   * Recovered all plaintext PDF passwords offline using the **Networkwalks Password Cracker** (default wordlist) and **John the Ripper** (`john --wordlist=/usr/share/wordlists/rockyou.txt`).

---

<a id="limitations-contraints-encountered"></a>
### 🧱 Limitations & Constraints Encountered

During the 5-day assessment, several operational boundaries, environmental safeguards, and tool configuration hiccups were encountered and managed:

1. **Rules of Engagement Scope Restrictions:**
   * Testing was strictly confined to `https://medirozahospital.com`. Subdomains discovered during passive OSINT (`ftp.medirozahospital.com`, `mail.medirozahospital.com`, `heartpatient.medirozahospital.com`) were documented for attack surface mapping but excluded from active exploitation per authorization limits.
   * Automated Denial of Service (DoS/DDoS), brute-force login flooding, physical security testing, and social engineering were strictly out of scope.

2. **Web Proxy Interception & HTTPS Inspection Setup:**
   * Initial form enumeration using Firefox Developer Tools (F12 Network Monitor) was effective for passive header inspection but lacked the ability to pause in-flight HTTP `POST` requests or dynamically re-send modified payloads.
   * Pivoting to **Burp Suite Community Edition** required manually configuring Firefox proxy routing (`127.0.0.1:8080`) and installing PortSwigger's local CA root certificate in Firefox to enable full HTTPS SSL/TLS traffic inspection on `https://medirozahospital.com` without browser security warnings.

3. **Shared Hosting Safeguards & Rate Limiting:**
   * Target infrastructure is hosted on Namecheap shared web servers (`199.188.201.16`) running LiteSpeed Web Server. To avoid triggering host-level firewall bans or ModSecurity connection blocks, automated dirbusting was avoided in favor of targeted manual directory auditing (`/robots.txt` discovery).

4. **OSINT API Key Dependencies:**
   * Passive scanning via `theHarvester` (`theHarvester -d medirozahospital.com -l 50 -b all`) returned missing API key notices for commercial databases (e.g., Censys, BeVigil). OSINT discovery successfully relied on free public search engines and DNS fallback queries without impacting assessment accuracy.

5. **Cryptographic Tool Pathing & Cache Hiccups:**
   * **`pdf2john` Path Issue:** Running `pdf2john` with relative directory paths (`../patient_report_3.pdf`) prepended relative directory slashes into the output hash string, causing John the Ripper to fail hash format parsing. Resolving this required navigating directly into the directory containing the `.pdf` file before executing `pdf2john`.
---

<a id="findings-and-exploitation"></a>
## 03. 🔍 Findings and Proof of Exploitation

<a id="finding-1"></a>
### 🚨 Finding 1: Unauthenticated Error-Based SQL Injection & Authentication Bypass

| Vulnerability Attribute | Details |
| :--- | :--- |
| **Vulnerability Title** | Unauthenticated Error-Based SQL Injection & Auth Bypass |
| **OWASP Category** | `OWASP WSTG-INPV-05` (Testing for SQL Injection) |
| **Vulnerable Endpoint** | `https://medirozahospital.com/patient/login.php` |
| **Vulnerable Parameter** | `username` (HTTP POST Body) |
| **CVSS v3.1 Rating** | `🔴 CRITICAL` (Score: `9.8`) — `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` |
| **Impacted Assets** | Patient Portal (`portal.php`), Electronic Health Records (EHR) |
| **Legal & Regulatory Risk**| Direct Breach of South African **POPIA** (§19 Security Measures) |

### 📌 Technical Description & Impact
An unparameterized error-based SQL Injection vulnerability exists within the authentication handling logic of `/patient/login.php`. User inputs submitted via the `username` HTTP POST parameter are concatenated directly into backend MySQL queries without input sanitization or prepared statements.

An unauthenticated remote attacker can exploit this flaw by submitting single quotes (`'`) to disrupt query syntax, triggering verbose PHP/MySQL error messages (`Warning: mysqli_query()`). By appending SQL comment delimiters (`-- `), an attacker can truncate backend password checks entirely (`admin' -- `), forcing the application to authenticate the session without valid credentials.

* **Primary Impact:** Full authentication bypass, session hijacking (`PHPSESSID`), unauthorized access to private patient pathology lab reports (**S. Dlamini**, **P. Reddy**, **E. Thompson**), and complete compromise of patient confidentiality under POPIA.

#### **Part 1: Initial Access to the 3 Encrypted PDF Patient Portal**

**Steps In Order:**
1. **Create an Evidence Directory:**

   Opened a Kali Linux VM terminal and created a dedicated workspace directory to organize all passive OSINT output logs:
   ```bash
   mkdir -p ~/week4-mediroza/step1 && cd ~/week4-mediroza/step1
   ```

2. **Search Certificate Transparency Logs (crt.sh):** Queried the public crt.sh database using curl with URL-encoded parameters to check for publicly registered subdomains without sending a single packet to the target web server:

    ```bash
   curl -s -G "https://crt.sh" --data-urlencode "q=%.medirozahospital.com"--data-urlencode "output=json" -o crtsh_raw.json
   grep -o '"name_value":"[^"]*"' crtsh_raw.json | sort -u | tee subdomains.txt
   ```

   **Command Breakdown:**
   * `curl -s -G`: Silent GET request to the `crt.sh` API endpoint.
   * `--data-urlencode "q=%.medirozahospital.com"`: Wildcard search query for subdomains ending in `.medirozahospital.com`.
   * `--data-urlencode "output=json"`: Requests structured JSON data output format.
   * `grep -o '"name_value":"[^"]*"' | sort -u | tee subdomains.txt`: Extracts domain strings, deduplicates entries, and outputs to terminal while saving to `subdomains.txt`.

**Findings:** The query returned zero publicly registered certificate records for subdomains.

---

3. **Run Multi-Source OSINT Enumeration (theHarvester):**
Executed `theHarvester` at 50 results per source to avoid third-party search engine rate-limiting and CAPTCHA blocks:

```bash
theHarvester -d medirozahospital.com -l 50 -b all -f harvester_mediroza
```

**Command Breakdown:**
* `-d medirozahospital.com`: Specifies the target domain.
* `-l 50`: Limits search results to 50 entries per source to ensure fast, unblocked execution.
* `-b all`: Instructs the tool to query all supported public search engines and OSINT modules simultaneously.
* `-f harvester_mediroza`: Automatically exports scan results to `harvester_mediroza.xml` and `harvester_mediroza.json`.

**Findings:** While paid API modules (such as Censys and BeVigil) displayed standard missing API key warnings, free DNS fallback discovered 3 active subdomains (`ftp.medirozahospital.com`, `heartpatient.medirozahospital.com`, and `mail.medirozahospital.com`), mapped the resolved server IP (`199.188.201.16`), and identified network Autonomous System Number `AS22612`.

4. **Query Public DNS & WHOIS Records:**
Queried public DNS resolvers for `A`, `NS`, and `MX` records using `dig`, and verified server IP ownership via `whois`:

```bash
dig medirozahospital.com A +short | tee /home/kali/week4-mediroza/step1/dns_records.txt
dig medirozahospital.com NS +short | tee -a /home/kali/week4-mediroza/step1/dns_records.txt
dig medirozahospital.com MX +short | tee -a /home/kali/week4-mediroza/step1/dns_records.txt
whois 199.188.201.16 | grep -iE "OrgName:|NetName:|OriginAS:" | tee /home/kali/week4-mediroza/step1/whois_summary.txt
```

**Findings:** Confirmed `medirozahospital.com` resolves directly to `199.188.201.16` on Namecheap shared web hosting (`NCNET-1` / `AS22612`) with standard Namecheap authoritative name servers (`dns1.namecheaphosting.com`), operating directly without a third-party cloud WAF or CDN.

---

### Task 2: Manual Web Reconnaissance & Source Code Inspection

5. **Inspect Server Headers and Metafiles:**
Opened Firefox inside Kali Linux with Developer Tools (Firefox-> More Tools-> Web Developer Tools) and visited `https://medirozahospital.com`, `/robots.txt`, and `/sitemap.xml`.

**Findings:** HTTP response headers confirmed LiteSpeed Web Server running `PHP/8.2.36`. Auditing `/robots.txt` disclosed three disallowed directories: `Disallow: /patient/`, `Disallow: /staff/`, and a hidden, unlinked `Disallow: /old/` directory.

6. **Inspect & Exfiltrate from the Exposed `/old/` Directory:**
Created the Step 3 workspace directory `/home/kali/week4-mediroza/step3-website-inspection/`:

```bash
mkdir -p /home/kali/week4-mediroza/step3-website-inspection
```

Navigated to `https://medirozahospital.com/old/` in Firefox, confirming open directory indexing displaying `mediroza_db_backup_2019.sql`. Saved the database backup file directly into `/home/kali/week4-mediroza/step3-website-inspection/mediroza_db_backup_2019.sql` and verified its structure in the terminal:

```bash
head -n 15 /home/kali/week4-mediroza/step3-website-inspection/mediroza_db_backup_2019.sql
```

**Findings:** The file was an unencrypted MySQL database backup (`mediroza_hr`) generated by Mediroza CMS 1.4.2 containing 30 internal employee salary records and 10 shareholder equity distribution records.

7. **Review Client-Side HTML Source Code:**
Opened `view-source:` in Firefox for `/index.html`, `/patient/login.php`, and `/staff/login.php`.

**Findings:** Both login forms submit `username` and `password` parameters via HTTP `POST` directly to `login.php` without CSRF protection tokens. `/patient/login.php` explicitly states: `"Sign in to view your lab reports and results."`

**Save a VirtualBox State Snapshot:**
Preserved all initial reconnaissance evidence by taking a clean VirtualBox snapshot named `Week4 Milestone 1-Steps1-to-3-Complete`.

### Task 3: Authentication Behavior Analysis & Transition to Burp Suite

8. **Test Baseline Login Behavior in Firefox DevTools:** Submitted dummy credentials (`testuser` / `testpassword`) on `/staff/login.php` and `/patient/login.php` while observing the Firefox Developer Tools Network tab.
    
    **Findings:** `/staff/login.php` returned a generic error message without issuing a session cookie. `/patient/login.php` issued a `**PHPSESSID**` session cookie and returned `*Username not found*`, confirming an account enumeration weakness (`**OWASP** **WSTG**-**IDNT**-04`).
    
9. **Pivot from Developer Tools to Burp Suite Community Edition:** Because Firefox DevTools lacked the capability to pause in-flight `**POST**` requests, modify raw body payloads dynamically, or manage session cookies cleanly across retries, testing pivoted to Burp Suite Community Edition.-   

10. **Configure Burp Suite Proxy Listener & **HTTPS** Interception:** Launched Burp Suite Community Edition in Kali Linux, verified the local Proxy Listener was running on `**127**.0.0.1:**8080**`, and configured Firefox manual proxy settings to route all traffic through `**127**.0.0.1:**8080**`. Imported PortSwigger's local CA root certificate into Firefox and confirmed clean **TLS** inspection on `[https://example.com`](https://example.com`) (**HTTP** **200** OK with **TLS** checkmark).


### Task 4: Error-Based SQL Injection Exploitation & Patient Portal Access

#### 1. Perform Single-Quote (`'`) Boundary Test

11. **Created the Step 5 workspace directory:** `/home/kali/week4-mediroza/step5-initial-access-of-patient-login/`:

```bash
mkdir -p /home/kali/week4-mediroza/step5-initial-access-of-patient-login
```

Entered a single apostrophe (`'`) into the **Username** field and `test` into **Password** on `/patient/login.php`.

* **Findings:** The server returned an explicit MySQL query error: `Warning: mysqli_query(): You have an error in your SQL syntax... near '''' at line 1`, confirming error-based SQL Injection (**OWASP WSTG-INPV-05**) via unsanitized dynamic string concatenation.

* **Note on Error Log Inspection:** Direct fetch attempts for `patient_error_log.txt` returned **HTTP 403 Forbidden** on the server; the log file `/home/kali/week4-mediroza/step5-initial-access-of-patient-login/patient_error_log.txt` was recorded locally during testing.

---

   #### 2. Payload Execution in Burp Repeater

   12. **Captured the login request in Burp Proxy HTTP history:** Sent it to Burp Repeater. Injected the authentication bypass payload:

   ```http
   username=admin%27+--+&password=test
   ```

The backend SQL query statement resolved to:

```sql
SELECT * FROM patients WHERE username='admin' -- ' AND password='...'
```

Clicked **Send**, receiving an `HTTP/2 302 Found` redirect response setting the header `Location: portal.php`.

---

#### 3. Session Cookie Continuity & Live Intercept Bypass

Because Burp Repeater and Firefox maintained two separate `PHPSESSID` session cookies, opening `portal.php` directly in Firefox redirected back to `login.php`. To authenticate Firefox's active browser session:

13. Enabled **Intercept ON** in Burp Proxy (`Proxy` -> `Intercept`).
14. Submitted `admin' -- ` directly in Firefox and clicked **Sign in**.
15.  Forwarded the intercepted `POST` request (`username=admin%27+--+&password=test`).

---

16. **Verify Portal Access & Retrieve Patient PDF Lab Reports:** Switched back to Firefox. Because Firefox transmitted its own active `PHPSESSID` cookie through the proxy during the 302 redirect, the browser successfully rendered the authenticated **My Reports** dashboard (`GET /patient/portal.php`). Downloaded all three patient PDF pathology lab reports directly into `/home/kali/week4-mediroza/step5-initial-access-of-patient-login/`:

-   `patient_report_1.pdf` (**S. Dlamini**)
- `patient_report_2.pdf` (**P. Reddy**)
-   `patient_report_3.pdf` (**E. Thompson**)

---

<a id="finding-2"></a>
### 🔐 Finding 2: Insecure Cryptographic Storage & Low-Entropy PDF Passwords (Milestone 2)

| Vulnerability Attribute | Details |
| :--- | :--- |
| **Vulnerability Title** | Insecure Cryptographic Storage & Low-Entropy PDF Passwords |
| **OWASP Category** | `OWASP WSTG-CRYP-01` (Testing for Weak Cryptography) |
| **Impacted Files** | `patient_report_1.pdf`, `patient_report_2.pdf`, `patient_report_3.pdf` |
| **CVSS v3.1 Rating** | `🟠 HIGH` (Score: `7.5`) — `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` |
| **Legal & Regulatory Risk** | Violation of South African **POPIA** (§19 Technical Safeguards) |

#### 📌 Summary of Findings
* **Online Encrypted File Retrieval:** Following the authentication bypass on `/patient/login.php`, 3 confidential patient pathology lab reports (`patient_report_1.pdf`, `patient_report_2.pdf`, `patient_report_3.pdf`) were downloaded directly from the active web portal in Firefox over HTTPS.
* **Dual-Method Hash Extraction:** For `patient_report_1.pdf` and `patient_report_2.pdf`, raw `$pdf$` encryption hashes were extracted using the **Networkwalks Hash Calculator**. For `patient_report_3.pdf`, the hash was extracted via the terminal using **`pdf2john`** and saved to `/home/kali/week4-mediroza/step6-third-patient-login-access/pdf3_hash.txt`.
* **Online Password Cracking:** All hashes were submitted to the **Networkwalks Password Cracker** online tool. Passwords for PDF 1 (**S. Dlamini**) and PDF 2 (**P. Reddy**) were cracked using the default built-in wordlist, while the VM terminal-extracted hash for PDF 3 (**E. Thompson**) was cracked using the **`rockyou.txt`** wordlist, unlocking all plaintext patient medical diagnostic records.

#### 💥 Impact of Finding 2
* **Loss of Patient Health Confidentiality:** Plaintext recovery of encrypted pathology lab reports directly compromises special personal information and private diagnostic health data for patients **S. Dlamini**, **P. Reddy**, and **E. Thompson**.
* **Failure of Technical Encryption Safeguards:** Relying on weak, low-entropy user passwords violates South Africa's **POPIA Section 19**, which mandates appropriate, reasonable technical measures to keep personal health information secure against unauthorized processing or access.
* **Reputational & Patient Trust Damage:** Exposure of medical diagnostic results damages patient trust and exposes Mediroza General Hospital to formal complaints and investigations.

---

#### **Part 2: Cryptographic Hash Extraction & Password Cracking**

17. **Hash Extraction & Password Cracking for PDF 1 & PDF 2:**
    * Uploaded `patient_report_1.pdf` and `patient_report_2.pdf` to the **Networkwalks Hash Calculator** to extract their `$pdf$` cryptographic hashes.
    * Copied the extracted hashes over to the **Networkwalks Password Cracker** to crack the passwords using the default built-in wordlist, successfully recovering the plaintext passwords for patients.

18. **Hash Extraction in Terminal & Online Cracking for PDF 3 (`rockyou.txt`):**
    * Navigated to the Step 6 workspace directory `/home/kali/week4-mediroza/step6-third-patient-login-access/` and extracted the `$pdf$` hash from `patient_report_3.pdf` via the terminal:
      ```bash
      cd /home/kali/week4-mediroza/step6-third-patient-login-access/
      pdf2john /home/kali/week4-mediroza/step5-initial-access-of-patient-login/patient_report_3.pdf > pdf3_hash.txt
      ```
    * Copied the extracted `$pdf$` hash string from `pdf3_hash.txt` in the terminal and pasted it into the **Networkwalks Password Cracker** online tool.
    * Uploaded/selected the custom **`rockyou.txt`** wordlist dictionary (`/usr/share/wordlists/rockyou.txt`) and clicked **START CRACKING**, successfully recovering the plaintext password to unlock and view patient **E. Thompson's** pathology lab report.

--- 
<a id="finding-3"></a>
### 📂 Finding 3: Unauthenticated Database Backup Exposure (Milestone 3)

| Vulnerability Attribute | Details |
| :--- | :--- |
| **Vulnerability Title** | Unauthenticated Open Directory Listing & SQL Backup Exposure |
| **OWASP Category** | `OWASP WSTG-CONF-05` (Review Old Backup and Unreferenced Files) |
| **Exposed Endpoint** | `https://medirozahospital.com/old/` |
| **Exfiltrated File** | `mediroza_db_backup_2019.sql` (Database: `mediroza_hr`) |
| **CVSS v3.1 Rating** | `🔴 CRITICAL` (Score: `9.1`) — `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` |
| **Legal & Regulatory Risk** | Severe Non-Compliance with South African **POPIA** (Mass PII & Financial Leakage) |

#### 📌 Summary of Findings
* **Online Reconnaissance & Directory Indexing:** Auditing `/robots.txt` in Firefox disclosed an unlinked directory (`Disallow: /old/`). Accessing `https://medirozahospital.com/old/` confirmed open directory indexing enabled on LiteSpeed Web Server, exposing `mediroza_db_backup_2019.sql`.
* **Direct Database Exfiltration:** The unencrypted MySQL database backup file (`mediroza_hr`, generated by Mediroza CMS v1.4.2) was downloaded over HTTPS directly into `/home/kali/week4-mediroza/step3-website-inspection/` without requiring any authentication or administrative credentials.
* **Mass PII, Payroll & Corporate Data Leakage:** Parsing the exfiltrated SQL file revealed complete internal staff records for 30 hospital employees (**Table A:** SA National ID numbers, phone numbers, email addresses, job titles, and monthly salaries up to R160,000/mo) alongside 100% of the hospital's corporate shareholder equity structure across 10 shareholders (**Table B:** 1,000,000 total shares).

#### 💥 Impact of Finding 3
* **Severe POPIA Data Breach & Regulatory Fines:** Unrestricted exposure of 30 employee South African National ID numbers, phone numbers, and salary figures represents a critical breach of **POPIA**, exposing the hospital to administrative fines up to **R10 million** and mandatory breach notification obligations.
* **Targeted Social Engineering & Executive Phishing:** Exfiltrated staff job titles, direct email addresses, and phone numbers provide threat actors with exact organizational mapping to launch highly targeted spear-phishing, business email compromise (BEC), and credential harvesting campaigns against hospital personnel.
* **Corporate Financial & Equity Exposure:** Leaking 100% of corporate shareholder equity distributions exposes confidential hospital ownership structures, share classes, and major trust stakes to public competitors and malicious external actors.

--- 
## 📊 Client Confidential Data


| Field | System Detail |
| :--- | :--- |
| **Backup Script Name** | `mediroza_db_backup_2019.sql` |
| **Data Owner & Scope** | Mediroza General Hospital *(Internal System Backup — Not for Public Export)* |
| **Source Software & Version** | Mediroza CMS v1.4.2 (Backup Module) |
| **Server Host & Database Name** | `localhost` |
| **Data Classification / Sensitivity** | 🔴 `HIGHLY CONFIDENTIAL` *(PII & Payroll Data — POPIA / HIPAA Subject)* |
| **Backup Timestamp** | `2019-08-27 02:14:03` |
 
---

### Table 1: Hospital Employee Salary & HR Inventory Data

| ID | Full Name | Job Title | Department | Monthly Salary (ZAR) | Date Joined |
| :-: | :--- | :--- | :--- | --: | :-: |
| **1** | **Dr. Rajesh Naidoo** | Chief Pathologist | `Diagnostics Lab` | **R 138,000** | 2009-03-16 |
| **2** | **Sarah Botha** | Chief Financial Officer | `Finance` | **R 152,000** | 2011-07-01 |
| **3** | **Dr. Johan van der Merwe** | Medical Director | `Management` | **R 160,000** | 2007-01-22 |
| **4** | **Dr. Anita Naicker** | Consultant Cardiologist | `Cardiology` | **R 132,000** | 2012-09-10 |
| **5** | **Dr. Ahmed Kara** | Consultant Physician | `Internal Medicine` | **R 128,000** | 2013-02-18 |
| **6** | **Dr. Yusuf Cassim** | Senior Registrar | `Emergency & Trauma` | **R 74,000** | 2018-05-04 |
| **7** | **Michael Roberts** | HR Director | `Human Resources` | **R 96,000** | 2010-11-15 |
| **8** | **Susan Pretorius** | HR Officer | `Human Resources` | **R 32,000** | 2016-08-23 |
| **9** | **Jameel Malik** | IT Systems Administrator | `IT` | **R 58,000** | 2015-04-12 |
| **10** | **Thabo Molefe** | Network Engineer | `IT` | **R 46,000** | 2017-10-02 |
| **11** | **Nomvula Khumalo** | Registered Nurse | `Emergency & Trauma` | **R 34,000** | 2016-01-19 |
| **12** | **Lerato Mokoena** | Registered Nurse | `Pediatrics` | **R 33,000** | 2017-06-07 |
| **13** | **Bongani Ndlovu** | Registered Nurse | `Cardiology` | **R 35,000** | 2015-12-01 |
| **14** | **Zanele Mahlangu** | Nursing Sister | `Theatre` | **R 47,000** | 2013-03-25 |
| **15** | **Kagiso Sithole** | Pharmacist | `Pharmacy` | **R 61,000** | 2014-09-08 |
| **16** | **Naledi Zulu** | Pharmacy Assistant | `Pharmacy` | **R 26,000** | 2019-02-14 |
| **17** | **Themba Nkosi** | Radiographer | `Radiology` | **R 44,000** | 2016-07-30 |
| **18** | **Palesa Radebe** | Radiographer | `Radiology` | **R 43,000** | 2017-04-11 |
| **19** | **Deepak Pillay** | Lab Technologist | `Diagnostics Lab` | **R 41,000** | 2015-05-20 |
| **20** | **Kavitha Govender** | Lab Technician | `Diagnostics Lab` | **R 35,000** | 2018-08-06 |
| **21** | **Dr. Suresh Moodley** | Consultant Radiologist | `Radiology` | **R 130,000** | 2012-02-28 |
| **22** | **Dr. Fatima Patel** | Pediatrician | `Pediatrics` | **R 118,000** | 2013-10-17 |
| **23** | **Nisha Singh** | Physiotherapist | `Rehabilitation` | **R 48,000** | 2016-11-09 |
| **24** | **Dr. Vikram Chetty** | Anaesthetist | `Theatre` | **R 135,000** | 2011-06-13 |
| **25** | **David Smith** | Facilities Manager | `Operations` | **R 52,000** | 2014-01-27 |
| **26** | **Karen O'Connor** | Billing Administrator | `Finance` | **R 29,000** | 2018-03-19 |
| **27** | **James Wilson** | Security Supervisor | `Operations` | **R 27,000** | 2019-09-02 |
| **28** | **Linda Fourie** | Receptionist | `Front Office` | **R 19,000** | 2020-02-10 |
| **29** | **Pieter van Wyk** | Procurement Officer | `Supply Chain` | **R 38,000** | 2015-08-24 |
| **30** | **Andile Mbeki** | Ward Clerk | `Administration` | **R 21,000** | 2019-11-05 |

---


#### Table 2: Hospital Shareholders (10 Shareholders)

| ID | Shareholder Name | Share Percentage (%) | Total Shares Held | Share Class |
| :---: | :--- | :---: | :---: | :---: |
| 1 | Dr. Rajesh Naidoo | 18.00% | 180,000 | Ordinary |
| 2 | Cedar Health Holdings (Pty) Ltd | 15.00% | 150,000 | Ordinary |
| 3 | Dr. Johan van der Merwe | 12.00% | 120,000 | Ordinary |
| 4 | Reddy Family Trust | 11.00% | 110,000 | Ordinary |
| 5 | Thabo Molefe | 10.00% | 100,000 | Ordinary |
| 6 | Sarah Botha | 9.00% | 90,000 | Ordinary |
| 7 | Dr. Ahmed Kara | 8.00% | 80,000 | Preferential |
| 8 | Naledi Zulu | 7.00% | 70,000 | Ordinary |
| 9 | Michael Roberts | 6.00% | 60,000 | Ordinary |
| 10 | Dr. Vikram Chetty | 4.00% | 40,000 | Preferential |

---
<a id="risk-rating"></a>
## 04. ⚖️ Risk Rating Matrix

Each vulnerability identified during the assessment is rated according to the **Common Vulnerability Scoring System (CVSS v3.1)** and evaluated against regulatory obligations under South Africa's **Protection of Personal Information Act (POPIA)**.

---

### 🚨 Finding 1: Unauthenticated Error-Based SQL Injection & Authentication Bypass
* **Severity Rating:** `🔴 CRITICAL`
* **CVSS v3.1 Base Score:** **9.8** (`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`) [2]
* **OWASP WSTG Category:** `OWASP WSTG-INPV-05` (Testing for SQL Injection) [2]
* **Target Endpoint:** `https://medirozahospital.com/patient/login.php` [2]
* **Risk Justification:** 
  This finding is classified as **CRITICAL** because an unauthenticated remote attacker on the public internet can bypass login security controls without valid user credentials or prior account access. The `username` HTTP POST parameter directly concatenates unsanitized user input into dynamic MySQL backend queries (`mysqli_query()`), generating explicit database syntax error warnings when probed with single quotes (`'`). Injecting the SQL comment bypass payload `admin' -- ` closes the string literal and comments out password verification entirely (`WHERE username='admin' -- ' AND password='...'`). Forwarding this payload in Burp Proxy issuing a 302 redirect establishes an authenticated session in Firefox, granting direct access to the `portal.php` dashboard and enabling the download of confidential medical pathology lab reports for **S. Dlamini**, **P. Reddy**, and **E. Thompson**. This represents a direct violation of technical security measures required under **POPIA Section 19**.

---

### 🔐 Finding 2: Insecure Cryptographic Storage & Low-Entropy PDF Passwords
* **Severity Rating:** `🟠 HIGH`
* **CVSS v3.1 Base Score:** **7.5** (`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`) [10]
* **OWASP WSTG Category:** `OWASP WSTG-CRYP-01` (Testing for Weak Cryptography) [10]
* **Target Assets:** `patient_report_1.pdf`, `patient_report_2.pdf`, `patient_report_3.pdf`
* **Risk Justification:**
  This finding is classified as **HIGH** because although the downloaded patient pathology reports are protected by PDF user password encryption, the underlying passphrases suffer from extremely low entropy. Extracted cryptographic `$pdf$` hashes for `patient_report_1.pdf` and `patient_report_2.pdf` were cracked using the **Networkwalks Password Cracker** with its default built-in wordlist. For `patient_report_3.pdf`, terminal hash extraction via `pdf2john` allowed cracking using the **`rockyou.txt`** dictionary. Recovering plaintext keys exposes confidential diagnostic health results, violating **POPIA** mandates regarding the protection of special personal health information.

---

### 📂 Finding 3: Unauthenticated Legacy Directory Exposure & Database Exfiltration
* **Severity Rating:** `🔴 CRITICAL`
* **CVSS v3.1 Base Score:** **9.1** (`CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`) [14, 15]
* **OWASP WSTG Category:** `OWASP WSTG-CONF-05` (Review Old Backup and Unreferenced Files) [16]
* **Target Endpoint & Asset:** `https://medirozahospital.com/old/` (`mediroza_db_backup_2019.sql`)
* **Risk Justification:**
  This finding is classified as **CRITICAL** due to the unauthenticated exposure of sensitive employee Personally Identifiable Information (PII) and corporate financial records. Inspecting `robots.txt` disclosed an unlinked path (`Disallow: /old/`) with directory indexing enabled on LiteSpeed Web Server, allowing direct HTTP download of `mediroza_db_backup_2019.sql`. Analysis of the backup revealed 30 staff records with South African National ID numbers, mobile numbers, email addresses, job titles, and monthly salaries up to R160,000/mo, as well as 100% of corporate shareholder equity distributions across 1,000,000 shares. Leaking National ID numbers and payroll data constitutes a severe **POPIA** data breach, exposing the hospital to large administrative fines.



---
<a id="recommendations-remediation"></a>
## 05. 🛡️ Recommendations and Remediation

To mitigate the critical security risks identified during this assessment, protect patient confidentiality, and ensure full compliance with South Africa's **Protection of Personal Information Act (POPIA)**, Mediroza General Hospital must implement the following actionable remediations.

---

### ⚡ Finding-Specific Tactical Remediations (Immediate Action Required — 24 to 48 Hours)

1. **Remediation for Finding 1:** Unauthenticated Error-Based SQL Injection (`/patient/login.php`)
* **OWASP Reference:** `OWASP WSTG-INPV-05` (Testing for SQL Injection)
* **Actionable Fix:** Replace all raw string concatenation in backend PHP code with parameterized prepared statements using PHP Data Objects (PDO) or MySQLi. This ensures user input in the `username` POST parameter is treated strictly as data rather than executable SQL code.

2. **Remediation for Finding 2:** Low-Entropy PDF Passwords (OWASP WSTG-CRYP-01)
OWASP Reference: OWASP WSTG-CRYP-01 (Testing for Weak Cryptography)
Actionable Fix: Upgrade PDF encryption from legacy PDF user passwords to AES-256 encryption standard.
Enforce High-Entropy Passphrases: Eliminate static, low-entropy default passwords for patient diagnostic lab reports. Implement dynamically generated, high-entropy passphrases (e.g., combining a cryptographically secure random string with patient-specific salt tokens) to render offline dictionary attacks (rockyou.txt) computationally infeasible.

3. **Remediation for Finding 3:** Unauthenticated Legacy Directory Exposure (/old/)
OWASP Reference: OWASP WSTG-CONF-05 (Review Old Backup and Unreferenced Files)
Actionable Fix: Immediately purge the exposed /old/ directory and permanently delete mediroza_db_backup_2019.sql from the public web server root (/home/kali/week4-mediroza/step3-website-inspection/).
Disable Directory Indexing: Disable open directory listing across the entire LiteSpeed web server configuration to prevent unlinked directories from exposing sensitive files.



### 🛡️ Strategic Security Roadmap (Long-Term Safeguards — 30 Days)

4. **Harden PHP Session Management Controls:**
    -   Enforce secure session cookie attributes across all patient and staff login portals:

```
session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'domain' => 'medirozahospital.com',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict'
]);
```

5. **Deploy Web Application Firewall (WAF) & Rate Limiting:**
    -   Configure LiteSpeed WebApp Firewall or ModSecurity rules to filter SQL injection signatures (`' OR 1=1`, `admin' --` ).-   Implement IP-based rate limiting on `/patient/login.php` and `/staff/login.php` to prevent automated authentication bypass probes.
    
6. **Establish POPIA Technical Safeguards & Continuous Auditing:**
    -   Mandate quarterly vulnerability scanning and annual external penetration testing aligned with **OWASP WSTG v4.2** guidelines to satisfy **POPIA Section 19** data protection obligations.

---
<a id="tools-and-resources"></a>
# 🔗 Tools & Resources

- **John the Ripper (Public Directory):** [https://distro.ibiblio.org/openwall/projects/john/1.9.0/](https://distro.ibiblio.org/openwall/projects/john/1.9.0/)
- **John the Ripper (Official Website):** [https://www.openwall.com/john/](https://www.openwall.com/john/)
- **Johnny GUI:** [https://openwall.info/wiki/john/johnny](https://openwall.info/wiki/john/johnny)
- **PDF Hash Extractor:** [https://www.onlinehashcrack.com/tools-pdf-hash-extractor.php](https://www.onlinehashcrack.com/tools-pdf-hash-extractor.php)
- **Networkwalks Hash Calculator:** [https://networkwalks.com/hash-calculator/](https://networkwalks.com/hash-calculator/)
- **Networkwalks Password Cracker:** [https://networkwalks.com/password-cracker/](https://networkwalks.com/password-cracker/)

- **7-Zip:** [https://7-zip.org/download.html](https://7-zip.org/download.html)

- **VirtualBox:** [https://virtualbox.org/wiki/Downloads](https://virtualbox.org/wiki/Downloads)
- **Kali Linux:** [https://kali.org/get-kali](https://kali.org/get-kali)
---