# Phishing Incident Analysis Report

---

> ⚠️ **Disclaimer:** This report is based on a simulated phishing scenario from TryHackMe's SOC Simulator (Introduction to Phishing). This is a training exercise conducted in a controlled environment, not a real-world incident.

---

## Report Information

| Field | Details |
|-------|---------|
| **Report Title** | Phishing Campaign Analysis — Multi-Vector Email Attack |
| **Analyst Name** | Rohan Singh Chauhan |
| **Platform** | TryHackMe SOC Simulator |
| **Scenario** | Introduction to Phishing |
| **SIEM Used** | Splunk |
| **Report Date** | June 24th, 2026 |
| **Classification** | TLP: WHITE (Training) |

---

## 1. Executive Summary

During this SOC simulation exercise, a total of **4 security alerts** were investigated across a corporate network environment. The investigation identified an active **multi-vector phishing campaign** targeting internal users via inbound emails containing malicious external links.

Out of 4 alerts:
- **3 were confirmed True Positives** — active phishing attempts
- **1 was confirmed False Positive** — legitimate onboarding email

One high-risk finding was identified: **Alert 8817** revealed that a user successfully accessed phishing infrastructure (firewall allowed the connection), posing a **high risk of credential theft**. Immediate escalation to L2/Incident Response was recommended.

**Overall Analyst Performance:**
| Metric | Result |
|--------|--------|
| True Positive Identification Rate | ✅ 100% |
| False Positive Identification Accuracy | ✅ 100% |
| Alerts Investigated | 4 |
| True Positives | 3 |
| False Positives | 1 |

---

## 2. Scope & Objectives

### Scope
- Alert queue monitoring via Splunk SIEM
- Phishing email header and link analysis
- Firewall log correlation
- True Positive / False Positive classification
- Case report documentation per alert

### Objectives
- Identify all active phishing threats in the alert queue
- Correctly classify each alert as True or False Positive
- Document findings and recommend remediation actions
- Escalate high-risk alerts appropriately

---

## 3. Tools & Environment

| Tool | Purpose |
|------|---------|
| **Splunk (SIEM)** | Log ingestion, search, and alert correlation |
| **Alert Queue** | Centralized alert triage and management |
| **Analyst VM** | Endpoint-level investigation |
| **Playbooks** | Standardized response procedures |
| **Case Reports** | Formal documentation of findings |
| **Firewall Logs** | Network traffic and connection analysis |

---

## 4. Alert Analysis

---

### 🟡 Alert ID: 8814
**Alert Rule:** Inbound Email Containing Suspicious External Link  
**Severity:** Medium  
**Type:** Phishing  
**Verdict:** ✅ FALSE POSITIVE

#### Investigation Summary
An inbound email was flagged due to containing an external link. Upon investigation, the following was determined:

**Analysis:**
- Investigated inbound onboarding email from domain `hrconnex.thm`
- Sender domain and embedded URL belong to the **same domain** — no mismatch detected
- No malicious attachments or suspicious content identified
- Splunk SIEM search for related indicators returned only the original email event
- No evidence of suspicious network activity or outbound connection attempts found

**Evidence:**
```
Sender Domain : hrconnex.thm
Embedded URL  : hrconnex.thm (same domain — consistent)
Attachments   : None
SIEM Results  : No related malicious indicators found
Network Logs  : No suspicious outbound connections
```

**Reason for False Positive Classification:**
The email was determined to be a **legitimate onboarding communication**. The link embedded in the email resolved to the same trusted domain as the sender, with no indicators of compromise present.

**Action Taken:** Alert closed as False Positive. No escalation required.

---

### 🔴 Alert ID: 8816
**Alert Rule:** Access to Blacklisted External URL Blocked by Firewall  
**Severity:** High  
**Type:** Firewall  
**Verdict:** 🚨 TRUE POSITIVE

#### Investigation Summary
The firewall detected and blocked an outbound connection attempt to a known malicious URL from an internal endpoint.

**Analysis:**
- Firewall triggered alert on outbound connection attempt to a blacklisted URL
- Connection originated from internal endpoint: `10.20.2.17`
- Destination URL matched known malicious indicators in threat intelligence feeds
- Security policy **"Blocked Websites"** successfully prevented the connection
- Correlation with phishing email alerts suggests this was a result of a user clicking a phishing link

**Evidence:**
```
Source IP     : 10.20.2.17 (Internal Endpoint)
Destination   : Blacklisted external URL
Firewall Rule : Blocked Websites (Security Policy)
Action        : BLOCKED ✅
Threat Intel  : URL confirmed malicious
```

**Impact Assessment:**
- User attempted to access a known malicious site
- No successful connection established — firewall block was effective
- Potential phishing campaign correlation identified

**Recommendation:**
- Escalate for review of user activity on `10.20.2.17`
- Correlate with phishing email alerts to identify campaign scope
- Review whether other endpoints attempted similar connections
- Consider user awareness training

**Action Taken:** Alert closed as True Positive. Escalated for user activity review on `10.20.2.17` and campaign correlation analysis with phishing email alerts.

---

### 🔴 Alert ID: 8815
**Alert Rule:** Inbound Email Containing Suspicious External Link  
**Severity:** Medium  
**Type:** Phishing  
**Verdict:** 🚨 TRUE POSITIVE

#### Investigation Summary
An inbound phishing email impersonating Amazon was identified, containing a shortened malicious URL.

**Analysis:**
- Email sender domain identified as `amazon.biz` — **typosquatting/impersonation** of legitimate Amazon domain (`amazon.com`)
- Email body contained a **shortened URL** (`bit.ly`) commonly used to obscure malicious destinations
- Message employed **urgency tactics** to pressure the recipient into immediate action
- Firewall logs confirmed an endpoint attempted to access the URL
- Connection was **blocked** by the "Blocked Websites" security rule

**Evidence:**
```
Sender Domain    : amazon.biz (Impersonating amazon.com)
Malicious URL    : bit.ly/[shortened link]
Tactics Used     : Urgency language, brand impersonation
Firewall Action  : BLOCKED ✅
Endpoint Activity: Attempted access confirmed in logs
```

**Attack Indicators (IOCs):**
| Indicator | Type | Description |
|-----------|------|-------------|
| `amazon.biz` | Malicious Domain | Amazon impersonation domain |
| `bit.ly/[link]` | Shortened URL | Obfuscated malicious destination |
| Urgency language | Social Engineering | Pressure tactics in email body |

**Impact Assessment:**
- User interacted with the phishing link
- Security controls (firewall) successfully prevented access to the malicious destination
- No confirmed data exfiltration

**Recommendation:**
- Block sender domain `amazon.biz` at email gateway
- Add `bit.ly` redirect destination to blocklist once resolved
- Notify affected user and conduct awareness briefing

**Action Taken:** Alert closed as True Positive. Recommended domain block for `amazon.biz` at email gateway and user awareness briefing for affected user.

---

### 🔴 Alert ID: 8817 — ⚠️ ESCALATION REQUIRED
**Alert Rule:** Inbound Email Containing Suspicious External Link  
**Severity:** Medium  
**Type:** Phishing  
**Verdict:** 🚨 TRUE POSITIVE — ESCALATION REQUIRED

#### Investigation Summary
A phishing email impersonating Microsoft was identified. Unlike other alerts, the user **successfully accessed the phishing infrastructure** — firewall allowed the connection, indicating a potential credential compromise.

**Analysis:**
- Email sender used **typosquatted domain** `m1crosoftsupport.co` impersonating Microsoft
- Email directed user to a **fake Microsoft login page** designed to harvest credentials
- Firewall logs show the connection was **ALLOWED** — user reached the phishing page
- This is the highest risk alert in this investigation — active credential theft risk

**Evidence:**
```
Sender Domain     : m1crosoftsupport.co (Typosquatted)
Impersonating     : Microsoft Support
Destination       : Fake Microsoft login page
Firewall Action   : ALLOWED ⚠️ (Connection Successful)
Credential Risk   : HIGH — User reached phishing page
```

**Attack Indicators (IOCs):**
| Indicator | Type | Description |
|-----------|------|-------------|
| `m1crosoftsupport.co` | Malicious Domain | Microsoft typosquatted domain |
| Fake login page | Phishing Infrastructure | Credential harvesting page |
| Urgency/authority language | Social Engineering | Microsoft impersonation |

**Impact Assessment:**
```
Severity  : Medium (High Risk due to successful connection)
Risk      : User accessed phishing page — credentials may be compromised
Firewall  : Failed to block — domain not yet in blocklist
Data Risk : Potential credential theft
```

**Recommendations:**
1. **Immediate:** Escalate to L2 / Incident Response team
2. **Immediate:** Reset affected user account password
3. **Immediate:** Enable MFA if not already active
4. **Short-term:** Add `m1crosoftsupport.co` to firewall blocklist
5. **Short-term:** Check if credentials were submitted to phishing page
6. **Short-term:** Search for other recipients of the same campaign
7. **Long-term:** User security awareness training on Microsoft impersonation attacks

**Action Taken:** Alert escalated to L2/IR team. Closed as True Positive — Escalation Required.

---

## 5. Campaign Overview — Connecting The Dots

After analyzing all 4 alerts, a pattern emerges suggesting a **coordinated phishing campaign**:

```
Attack Flow Identified:
─────────────────────────────────────────────────────
Step 1 → Phishing emails sent to internal users
         (Alert 8815 — Amazon, Alert 8817 — Microsoft)
         
Step 2 → Users click malicious links in emails
         
Step 3 → Firewall detects outbound connections
         (Alert 8816 — Blacklisted URL blocked)
         
Step 4 → Alert 8817 — One user successfully
         reached phishing infrastructure
         (Firewall did not block — new domain)
─────────────────────────────────────────────────────
```

**Campaign Characteristics:**
| Attribute | Detail |
|-----------|--------|
| Attack Type | Phishing — Brand Impersonation |
| Brands Impersonated | Amazon, Microsoft |
| Techniques Used | Typosquatting, URL shortening, Urgency tactics |
| Target | Internal corporate users |
| Goal | Credential harvesting |
| Success Rate | Partial — 1 user reached phishing page |

---

## 6. Indicators of Compromise (IOCs)

| IOC | Type | Severity | Alert |
|-----|------|----------|-------|
| `amazon.biz` | Malicious Domain | Medium | 8815 |
| `bit.ly/[redirect]` | Shortened URL | Medium | 8815 |
| `m1crosoftsupport.co` | Malicious Domain | Medium | 8817 |
| `10.20.2.17` | Internal IP | High | 8816 |
| Blacklisted external URL | Malicious URL | High | 8816 |

---

## 7. MITRE ATT&CK Mapping

| Technique | ID | Description |
|-----------|-----|-------------|
| Phishing | T1566.001 | Spearphishing via email link |
| Masquerading | T1036 | Domain typosquatting |
| Obfuscated Files | T1027 | URL shortening to hide destination |
| Credential Phishing | T1056 | Fake login page for credential harvest |

---

## 8. Summary & Recommendations

### Immediate Actions Required
```
🚨 HIGH     → Reset credentials for user who accessed Alert 8817
🚨 HIGH     → Block m1crosoftsupport.co at firewall/email gateway
🚨 HIGH     → Investigate 10.20.2.17 for further compromise
⚠️ MEDIUM   → Block amazon.biz at email gateway
⚠️ MEDIUM   → Notify all affected users
```

### Long-Term Recommendations
```
1. Email Gateway Enhancement
   → Implement stricter domain spoofing detection
   → Add typosquatting detection rules

2. User Awareness Training
   → Phishing simulation exercises
   → How to identify brand impersonation
   → Reporting suspicious emails procedure

3. Firewall Rules
   → Regular threat intelligence feed updates
   → Faster blacklist propagation

4. Incident Response
   → Establish faster L1 → L2 escalation path
   → Define credential compromise response playbook
```

---

## 9. Analyst Performance

| Metric | Result |
|--------|--------|
| Total Alerts Investigated | 4 |
| True Positives Identified | 3/3 ✅ |
| False Positives Identified | 1/1 ✅ |
| True Positive Rate | **100%** |
| False Positive Identification Accuracy | **100%** ✅ |
| High Risk Alert Escalated | ✅ Yes (8817) |
| Investigation Status | All Alerts Resolved ✅ |

---

## 10. Conclusion

This simulation demonstrated a realistic multi-vector phishing campaign targeting internal users through brand impersonation of Amazon and Microsoft. The investigation successfully identified all true threats while correctly clearing the legitimate onboarding email as a false positive.

The highest risk finding — **Alert 8817** — highlighted a scenario where a user reached live phishing infrastructure, representing a real-world credential theft risk that required immediate L2 escalation and account remediation.

**Key Takeaway:**
> Not all phishing attacks are blocked by technical controls. Alert 8817 proved that even when emails bypass initial filters, SOC analysts must correlate firewall logs, SIEM data, and email analysis to identify successful phishing attempts before credentials are compromised.

---

## Badges Earned

| Badge | Achievement |
|-------|------------|
| 🔍 First Alert Closed | Successfully triaged and closed first SOC alert |
| 🎮 First Scenario Completed | Completed Introduction to Phishing scenario |
| 🎯 100% True Positive Rate | Zero missed threats across all alerts |

---

*Report prepared by: Rohan Singh Chauhan*  
*Platform: TryHackMe SOC Simulator*  
*Scenario: Introduction to Phishing*  
*Date: June 24th, 2026*  

---

> 🔗 **TryHackMe Profile:** [tryhackme.com/p/RohanSinghChauhan](https://tryhackme.com/p/RohanSinghChauhan)  
> 🔗 **LinkedIn:** [linkedin.com/in/rohan-singh-chouhan-962434327](https://www.linkedin.com/in/rohan-singh-chouhan-962434327)  
> 🔗 **GitHub:** [github.com/rohansinghh01](https://github.com/rohansinghh01)
