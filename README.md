# Phishing Email Analysis & Threat Investigation

> **Portfolio lab / simulated case.** The included email is synthetic and uses reserved example domains. It is not a real phishing email and contains no live malicious URL or attachment.

## Objective
Practice a repeatable phishing-triage workflow: preserve evidence, inspect message headers, review sender authentication, extract indicators of compromise (IOCs), check public threat-intelligence sources safely, assess social-engineering clues, and write a defensible incident report.

## Tools
- **MXToolbox** — header analysis and DNS/authentication lookups (free features; not open-source)
- **VirusTotal** — reputation lookups for public, non-sensitive indicators
- **urlscan.io** — website/redirect investigation when a URL is safe and appropriate to submit
- **Git and GitHub** — version control and portfolio publication

This project does **not** use Splunk.

## Case details
- Case ID: `PHISH-001`
- Sample: `BRADESCO LIVELO.eml`
- Status: Training simulation; findings must be completed by the analyst
- Scope: Email metadata, sender/authentication clues, URL structure, social engineering, IOC recording, and recommendations
- No live website should be opened directly from an untrusted email.


## Workflow
1. **Preserve the sample.** Keep the original `.eml` unchanged; work on a copy. Record when and where the sample was obtained. This sample is synthetic.
2. **Review headers.** In a real case, inspect `From`, `Reply-To`, `Return-Path`, `Received`, `Message-ID`, and `Authentication-Results`. Compare display name and actual sender domain. Do not infer maliciousness from one field alone.
3. **Check authentication.** Use MXToolbox or the receiving mail provider's trusted header results to review SPF, DKIM, and DMARC. A pass does not prove an email is safe; a fail can have benign causes.
4. **Extract and assess URLs.** Compare the visible link text with the actual target, identify redirects/shorteners, look for lookalike domains and unusual subdomains. Do not click the link.
5. **Check reputation carefully.** Search only public, non-sensitive indicators in VirusTotal. If using urlscan.io, select visibility deliberately. Public scans can expose submitted URLs and scan details. Never submit personal, authenticated, confidential, or tokenized URLs. See the service documentation linked below.
6. **Record evidence.** For every claim, record the tool, query/indicator, date/time (UTC if possible), result, screenshot filename, and analyst interpretation.
7. **Make a verdict.** Use `Phishing`, `Suspicious`, `Benign`, or `Inconclusive` and explain confidence. Separate observed facts from interpretation.
8. **Write the report.** Copy `incident-report.md` and replace placeholders with actual results. Do not invent results.
9. **Publish safely.** Remove personal data, real email addresses, internal hostnames, authentication tokens, private URLs, API keys, and confidential message content before pushing to GitHub.

## Public-service privacy cautions
- VirusTotal says queried/submitted indicators can be added to its dataset and become accessible to the community. Do not submit sensitive or personally identifiable information: https://docs.virustotal.com/docs/searching
- urlscan.io scans can be public, unlisted, or private. Public scans are visible publicly; choose visibility based on sensitivity: https://docs.urlscan.io/pages/visibility
- Prefer already-existing reputation results for safe, public indicators. Do not submit a private organization's URLs or an employee's unique tracking links.

## Evidence and screenshots
Save screenshots in `screenshots` with descriptive names.

## Limitations
- The bundled sample is a simulation; its header fields are illustrative, not evidence of a real sender or real SPF/DKIM/DMARC results.
- Reputation tools can disagree or have no data. “No detections” does not prove safety.
- This is a learning project, not a substitute for an organization's incident-response process.

## References
- MXToolbox: https://mxtoolbox.com/
- VirusTotal searching and privacy: https://docs.virustotal.com/docs/searching
- urlscan.io visibility: https://docs.urlscan.io/pages/visibility
- urlscan.io documentation: https://docs.urlscan.io/
- CyberChef: https://gchq.github.io/CyberChef/


## Investigation Findings

### 1. Email Header Analysis
The sender's email address, Return-Path, recipient address and relevant sender IP information were identified from the email headers. These values were recorded as part of the evidence collection process.
<img width="1920" height="1002" alt="HeaderDetails" src="https://github.com/user-attachments/assets/59d4ca05-27ad-42b4-ae86-8b70eacd277e" />
<img width="1645" height="379" alt="Header values" src="https://github.com/user-attachments/assets/3cdbd690-052b-44fa-94f7-58da6f61e72c" />

### 2. IP Reputation Analysis
The identified IP address was checked using the Blacklist Check feature on MXToolbox. The lookup reported that the IP was blacklisted.
<img width="1645" height="379" alt="Senders ip" src="https://github.com/user-attachments/assets/893a0651-9e9f-42fb-adcf-f9b4d0ff3280" />

This finding indicates a potential reputation concern. The specific blacklist provider, listing reason and lookup timestamp should be retained as evidence. A blacklist result alone does not conclusively establish that the email is malicious.
<img width="1551" height="525" alt="Senders ip rerutation" src="https://github.com/user-attachments/assets/a06fec58-b0a6-4f16-95b9-9eae379d1038" />

### 3. SPF Authentication Analysis
The SPF authentication check performed using MXToolbox returned a `fail` result.

This indicates that the sending IP did not satisfy the evaluated domain's SPF policy. The result should be interpreted alongside the original email headers and the relevant domain's SPF configuration.
<img width="1920" height="997" alt="SPF_Authentication" src="https://github.com/user-attachments/assets/e0314a11-16f2-452e-9489-fb7172cd5ebe" />

### 4. Hyperlink Inspection
The email contained a clickable link associated with the expiring-loyalty-points message. Hovering over the link revealed a destination different from the link's displayed text or apparent destination.

This discrepancy was documented as a potential indicator of URL obfuscation or redirection. The destination must be assessed separately before it can be classified as malicious.
<img width="1909" height="1030" alt="Email" src="https://github.com/user-attachments/assets/435bf416-6a0f-459e-8710-7524d2c15eb6" />

### 5. Overall Assessment
The investigation identified multiple indicators requiring further scrutiny: a reported IP blacklist listing, an SPF failure, and a potentially misleading hyperlink.

These observations warrant a suspicious-email assessment. The final phishing classification and confidence level should be supported by the exact destination URL, the blacklist listing details, and any additional corroborating evidence.
