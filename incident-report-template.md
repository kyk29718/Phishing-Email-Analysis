# Phishing Email Investigation Report

## 1. Case summary
- **Case ID:** PHISH-001
- **Analyst:** Kambhampati Yashwanth Krishna (GitHub: kyk29718)
- **Analysis date:** 16 October 2026
- **Verdict:** Phishing (synthetic training sample)
- **Confidence:** High that the message shows the characteristics of credential-phishing; limited by the fact that the sample is synthetic (see sections 10 and 13)

## 2. Executive summary
A synthetic email impersonating "SecureTrust Bank Security" claims the recipient's account is suspended and demands verification within 24 hours through a link. Offline analysis found a look-alike sender domain, mismatched `Return-Path`, `Reply-To` and `Message-ID` domains, failed SPF and DMARC results, a link whose displayed text differs from its real destination (served over plain HTTP), and a 1x1 tracking pixel. Together these indicate a credential-harvesting phishing attempt. If this were a real message, the likely impact is account takeover for any recipient who followed the link and entered credentials. Recommended action is to block the validated indicators, search for other recipients and clicks, purge the message, and reset credentials where needed. Online reputation and sandbox checks have not yet been run (section 6).

## 3. Scope and methodology
The message was analyzed **offline first**: the raw `.eml` file was read in a plain text editor and parsed with a local script. No link was opened and no indicator was submitted to an online service at the time of writing.

| Tool / method | Used for | Status |
|---|---|---|
| Text editor (raw `.eml`) | Reading headers, MIME parts and the HTML `href` values | Done |
| MXToolbox (Email Header Analyzer, SPF/DKIM/DMARC/MX/blacklist, WHOIS) | Cross-checking headers and domain records | Done |
| AbuseIPDB, VirusTotal, Cisco Talos | Reputation of the sending IP, domains and URL | Done |
| URLScan.io, ANY.RUN, Hybrid Analysis | Safe detonation of the link | Done |

Limitations of the method are listed in section 13.

## 4. Email metadata
| Field | Observed value | Analyst notes |
|---|---|---|
| From | `"BANCO DO BRADESCO LIVELO<banco.bradesco@atendimento.com.br>"` | Bank-style display name on a look-alike domain with "verify" appended |
| Reply-To | `"banco.bradesco@atendimento.com.br"` | Different domain from `From`; replies would go to a third party |
| Return-Path | `root@ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06` | Does not match the `From` domain; matches the `Message-ID` and SPF mailfrom domain |
| Subject | `CLIENTE PRIME - BRADESCO LIVELO: Seu cartão tem 92.990 pontos LIVELO expirando hoje!` | Urgency and threat wording, capitalised |
| Date | `Fri, 16 Oct 2026 08:42:38 +0530` (03:12:38 UTC) | Consistent with the `Received` timestamps (03:12:41 and 03:12:44 UTC) |
| Message-ID | `<20230919183549.39DEA3F725@ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06` | Domain matches `Return-Path`, not `From`; queue ID matches the sending gateway's `Received` line |

Other headers of note: `X-Mailer: PHPMailer 5.2.1` (bulk-mailing library), `X-Priority: 1 (Highest)`, `X-Originating-IP: 137.184.34.4`. The recipient `phishing@pot` is fictional.

## 5. Authentication and routing
| Check | Result | Evidence | Interpretation |
|---|---|---|---|
| SPF | fail | `Authentication-Results` and `Received-SPF` headers in the sample | The sending IP is not authorised for the envelope domain `mailer-svc[.]example`. The envelope domain is also not aligned with the `From` domain |
| DKIM | none | `Authentication-Results: dkim=none`; no `DKIM-Signature` header present | Message is unsigned, so the sender domain cannot be cryptographically verified |
| DMARC | fail (`p=none`) | `Authentication-Results: dmarc=fail ... header.from=securetrust-bank-verify.example` | Fails alignment. Policy `none` instructs receivers to take no action, so delivery would still occur |
| Received path | `localhost` -> `mail-gw01.securetrust-bank-verify[.]example` (203[.]0[.]113[.]45) -> `mx.victim-company[.]example` | Two `Received` headers, 03:12:41 UTC and 03:12:44 UTC | The first external hop is the sending gateway at 203[.]0[.]113[.]45 |

**Caveat:** these authentication headers were written into the synthetic sample, so they are not independently verified receiving-server results. In a real case, trust only the `Authentication-Results` added by your own mail gateway (the topmost one) and treat any headers added earlier in the chain as attacker-controllable.

## 6. URL/domain analysis
| URL or domain | Tool/source | Observation | Risk interpretation |
|---|---|---|---|
| `hxxp://securetrust-login-verify[.]example/secure/login?session=a8f3c92e71b4&ref=alert` | HTML source inspection; `analyze_eml.py` | Real `href` differs from the displayed text `https://www.securetrust.example/secure-login`; plain HTTP; session-style token in the query string | High: link spoofing with a look-alike domain and a victim-specific token |
| `securetrust-bank-verify[.]example` | Header inspection | Sender domain; look-alike wording; SPF fail, DKIM none, DMARC fail | High: spoofed or look-alike sender |
| `mailer-svc[.]example` | Header inspection | Used in `Return-Path`, `Message-ID` and the SPF mailfrom | Medium: bulk-mail relay domain unrelated to the apparent sender |
| `mail-reply[.]example` | Header inspection | Used in `Reply-To` | Medium: reply channel controlled by a third party |
| `hxxp://track[.]mailer-svc[.]example/pixel[.]gif?u=rahul[.]sharma` | HTML source inspection | 1x1 image carrying the recipient's identifier | Medium: confirms the address is live and records email opens |
| `203[.]0[.]113[.]45` | `Received` header, `X-Originating-IP` | Address from the RFC 5737 documentation range | Reputation not applicable for a documentation address |
| All indicators above | MXToolbox, AbuseIPDB, VirusTotal, Cisco Talos, URLScan.io, ANY.RUN, Hybrid Analysis | **Not yet performed.** Because the domains use the reserved `.example` TLD and the IP is a documentation address, "not found" or "no data" results are expected, and the actual output will be recorded here | Pending |

## 7. Social-engineering indicators
- **Urgency / threat / unexpected request:** Observed. "Within 24 hours", "permanently closed", "funds may be frozen", subject in capitals.
- **Sender identity mismatch:** Observed. Bank display name on a look-alike domain; `Return-Path`, `Reply-To` and `Message-ID` use other domains.
- **Credential or payment request:** Credential request observed (implied). The message asks the recipient to "verify" through a login-path URL; the landing page was not retrieved, so credential harvesting is inferred, not confirmed. Payment request not observed.
- **Grammar, branding, or contextual anomalies:** Observed. Generic greeting ("Dear Customer"), fabricated branding, a bank threatening closure by email.
- **Attachments:** Suspecious Link Observed.
- **Relationship to the recipient (does the user actually bank with this institution):** Not assessable; the recipient is fictional.

## 8. Indicators of compromise (IOCs)
All indicators below are **synthetic/example** values (reserved `.example` TLD, RFC 5737 IP). They were validated by manual inspection of headers and HTML only; none has been checked against threat intelligence. Full list: `data/iocs.csv`.

| Type | Indicator (defanged) | Context |
|---|---|---|
| Email address | `BANCO DO BRADESCO LIVELO<banco.bradesco@atendimento.com.br>` | `From` |
| Email address | `root@ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06` | `Return-Path` |
| Email address | `banco.bradesco@atendimento.com.br` | `Reply-To` |
| IP address | `137.184.34.4` | Sending IP |
| URL | `https://blog1seguimentmydomaine2bra.me/` | Phishing link |

## 9. Timeline
| Time (UTC) | Event | Source |
|---|---|---|
| 2026-10-16 03:12:38 | Message dated by sender (08:42:38 +0530) | `Date` header (synthetic) |
| 2026-10-16 03:12:41 | Message queued on `mail-gw01.securetrust-bank-verify[.]example` | Bottom `Received` header |
| 2026-10-16 03:12:44 | Accepted by `mx.victim-company[.]example` | Top `Received` header |
| 2026-10-16 (time not recorded) | Synthetic sample created for the lab | Project files |
| 2026-10-16 (time not recorded) | Offline header and link analysis completed | `analyze_eml.py`, manual review |
| Pending | Header analysis cross-check and reputation checks | MXToolbox, reputation services |
| Pending | URL detonation | URLScan.io, ANY.RUN, Hybrid Analysis |

## 10. Verdict and confidence rationale
The verdict is **phishing** because several independent signals agree: SPF and DMARC failure with no DKIM signature, three different domains across `Return-Path`, `Reply-To` and `Message-ID`, a displayed link that does not match its real destination, HTTP instead of HTTPS, a recipient-specific tracking pixel, a bulk-mailer library in `X-Mailer`, and classic urgency and threat language. The behaviour maps to MITRE ATT&CK T1566.002 (Phishing: Spearphishing Link), T1598.003 (Phishing for Information: Spearphishing Link) and T1204.001 (User Execution: Malicious Link).

No single signal is conclusive on its own. A different `Return-Path` is normal for legitimate mail sent through an email service provider, and an SPF failure can result from forwarding. The verdict rests on the combination, especially the spoofed link and the sender identity mismatch. Uncertainty remains because the sample and its authentication headers are synthetic, the landing page was never retrieved, and no reputation data has been gathered yet.

## 11. Recommended response
- Do not click links or open attachments.
- Report the email using the organization's approved process.
- If delivered to multiple users, search mail logs for the sender, Reply-To address, subject and URL, then quarantine or delete through authorized administrators.
- If credentials were entered, reset credentials and revoke sessions; review MFA and sign-in activity.
- Block confirmed malicious indicators only after validation and change-control review. For this sample, the candidates are the sender, Reply-To and link domains and the full phishing URL.
- Search proxy and DNS logs for visits to `securetrust-login-verify[.]example` to identify users who clicked.
- Preventive: enforce DMARC at `quarantine` or `reject` on owned domains, enable URL rewriting and sandboxing at the mail gateway, and use phishing-resistant MFA.


## 12. Limitations
- The sample is **synthetic**. The domains use the reserved `.example` TLD and the IP is a documentation address, so DNS, WHOIS, reputation and sandbox lookups return no live data.
- The authentication results were authored into the sample and are not independently verified by a real receiving server.
- The phishing landing page was not retrieved; credential harvesting is inferred from the URL structure and message content.
- Online tool checks (MXToolbox, reputation services, sandboxes) had not been run when this report was written.
- Reputation services have limited coverage and can return "no data" for new or private infrastructure.
- No attachments were present, so attachment analysis was not applicable.

