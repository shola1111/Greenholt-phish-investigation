# Phishing Incident Investigation: "Greenholt Phish"

## Scenario / Objective
A Sales Executive at Greenholt PLC reported a suspicious email from a known customer containing an unexpected money transfer request and an unsolicited attachment. As a Level 1 SOC Analyst, my objective was to analyze the email headers, investigate the sender's infrastructure, and analyze the attachment to determine if this was a legitimate request or a phishing attempt.

## Tools Used
*   **Mozilla Thunderbird:** Used to open the `.eml` file, view the raw message source, and analyze email headers.
*   **WHOIS Lookup:** For identifying the owner of the originating IP.
*   **SPF/DMARC Lookup Tools:** For verifying email authentication.
*   **Linux CLI (`sha256sum`):** For hashing the malicious file.
*   **VirusTotal:** For malware analysis and file type verification.

---

## Investigation Steps

### 1. Email Header & Artifact Extraction
I opened the provided email sample in **Mozilla Thunderbird**. Using the "View Source" feature, I examined the raw email headers and body to extract key artifacts and identify red flags.
*   **Sender Display Name:** `Mr James Jackson` (Impersonating a known customer).
*   **Sender Email Address:** `info@mutawamarine[.]com`
*   **Transfer Reference Number:** `09674321`
*   **Originating IP Address:** `192[.]119[.]71[.]157`
*   *Analysis:* The email used a generic greeting and requested a money transfer, which are classic social engineering tactics.

### 2. Infrastructure & Authentication Analysis
I investigated the originating IP address and the domain's email authentication records to verify legitimacy.
*   **IP WHOIS Lookup:** I ran a WHOIS search on `192[.]119[.]71[.]157`. The IP is owned by **HostPapa**, a commercial hosting provider. Attackers frequently use commercial hosting services to launch phishing campaigns.
*   **SPF Record Check:** I checked the return-path domain (`mutawamarine@mail[.]com`). The full SPF record is `v=spf1 include:spf.protection.outlook.com -all`. 
    *   *Note:* While an SPF record exists, the use of a lookalike domain (`mail.com` instead of the real company domain) indicates a spoofing attempt.
*   **DMARC Lookup:** I performed a DMARC lookup for the return-path domain. The complete record is `v=DMARC1; p=quarantine; fo=1`.

### 3. Attachment Analysis (Static Analysis)
The email contained an unsolicited attachment. I extracted it in a safe, isolated environment.
*   **Attachment Name:** `SWT_#09674321____PDF__.CAB` (Using a `.CAB` extension to hide the true file type).
*   **SHA256 Hash Extraction:** Using the Linux command `sha256sum`, I generated the hash of the file:
    `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f`
*   **VirusTotal Investigation:** I submitted the SHA256 hash to VirusTotal.
    *   **File Size:** `400.26 KB`
    *   **Actual File Type:** **`RAR`** (A compressed archive file).
    *   *Analysis:* The attacker disguised a RAR archive as a PDF by renaming the extension to `.CAB`. This is a common evasion technique used to bypass email filters and trick users into extracting malware.

---

## Indicators of Compromise (IOCs)

*   **Sender IP:** `192[.]119[.]71[.]157`
*   **Sender Email:** `info@mutawamarine[.]com`
*   **Malicious Attachment Name:** `SWT_#09674321____PDF__.CAB`
*   **File Hash (SHA256):** `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f`
*   **Hosting Provider:** HostPapa

## Conclusion & SOC Verdict
**Verdict: Malicious (True Positive).**

This email is a confirmed phishing and Business Email Compromise (BEC) attempt. The attacker spoofed a known customer (Mr. James Jackson) and used a lookalike domain to request a fraudulent money transfer. They attempted to bypass security controls by disguising a malicious RAR archive as a PDF file. 

**SOC Recommendation:** 
*   Block the sender IP (`192[.]119[.]71[.]157`) and domain at the email gateway.
*   Quarantine the email from the user's inbox and search for other recipients.
*   Add the SHA256 hash to the endpoint protection blocklist.
*   Notify the Sales Executive that the email was indeed malicious and to verify any future money transfers via phone call.
