# THM Write-Up: SOC L1 Alert Reporting & Escalation

**Platform:** TryHackMe  
**Path:** SOC Level 1  
**Difficulty:** Easy  
**Date:** 2025

---

## Overview

After triaging an alert, an L1 analyst needs to do two things: write a clear report of their findings, and decide whether to escalate. This room introduces the formal process for both — including the Five W's reporting template, escalation criteria, and SOC communication rules during incidents.

---

## Task 1 — Introduction

Access to the SOC SIEM Dashboard is granted to complete the tasks in this room.

---

## Task 2 — Alert Funnel

Most alerts are handled entirely by L1 analysts and closed as false positives. The ones that require deeper investigation get escalated to L2. Think of it as a funnel — a large volume of alerts enters, and only a fraction makes it through to L2.

Two key terms introduced here:

- **Alert Reporting** — Writing a concise, structured summary of an alert and your findings. For true positives, this report is what L2 analysts use to continue the investigation — they don't start from zero.
- **Alert Escalation** — The act of passing a confirmed or suspected true positive to an L2 analyst for deeper analysis.

### Questions

**Q: What is the process of passing suspicious alerts to an L2 analyst?**  
**A: Alert Escalation**

**Q: What is the process of formally describing alert details and findings?**  
**A: Alert Reporting**

---

## Task 3 — Reporting Guide

Proper alert reports follow the **Five W's** template:

| W | Field | Description |
|---|---|---|
| **What** | Action/Event | What exact action or event sequence occurred |
| **When** | Time | When did the suspicious activity start and end |
| **Who** | User/Actor | Which user or system caused the alert |
| **Where** | Location | Which device, IP, or location was involved |
| **Why** | Verdict Reasoning | Why you reached your final verdict |

### Questions

**Q: Which user email leaked the sensitive document?**  
**A: m.boslan@tryhackme.thm**

Found by locating the "Sensitive Document Share to External" alert and checking the Source User Email field.

**Q: Who is the sender of the suspicious phishing email?**  
**A: support@microsoft.com**

Found in the newest alerts queue under the phishing alert's sender field. This address is spoofed — details below.

**Q: Using the Five W's template, what flag did you receive after writing a good report?**  
**A: THM{nice_attempt_faking_microsoft_support}**

The report was written as follows:

---

**What:** Phishing Email

**When:** March 27th, 2025 at 19:25

**Who:** `support@microsoft.com` (spoofed — impersonating Microsoft Support)

**Where:** Delivered to Eddie Huffman, IT Manager `<e.huffman@tryhackme.thm>`

**Why:** Several indicators confirm this is a phishing attempt:

- **Spoofed Sender** — Microsoft does not initiate contact through a generic support email, and `support@microsoft.com` is not an official Microsoft support address
- **SPF Fail** — Sender Policy Framework check failed, meaning the email was not sent from a server authorised to send on behalf of microsoft.com
- **DKIM Fail** — DomainKeys Identified Mail check failed, meaning the email was not cryptographically signed by the domain. Both authentication checks failing confirms the sender is spoofed.
- **Social Engineering** — The email uses urgency tactics ("600% price increase", "urgent notice", "download the report") to pressure the recipient into acting without thinking
- **Suspicious Attachment** — A `REPORT.rar` file was attached, a common delivery method for malware in phishing campaigns

---

## Task 4 — Escalation Guide

After writing a report, the next decision is whether to escalate to L2. Escalate when:

- The alert looks like a **true positive** or active attack
- The potential impact is **high** (data loss, malware, privilege abuse)
- **Deeper investigation** is needed beyond L1 scope
- You are **not confident** in your conclusion and need a second opinion
- The activity affects **multiple users or hosts**, or is repeating

To escalate, assign the alert to an available L2 analyst. A formal escalation note may also be required depending on your SOC's process.

### Alert 1 — Phishing Email Escalation

Going back to the phishing alert from Task 3:

1. Status changed to **In Progress**
2. Verdict set to **True Positive**
3. Assigned to **E.Fleming (L2)**

**Flag:** `THM{good_job_escalating_your_first_alert}`

---

### Alert 2 — Reverse Shell via Web Shell (High)

The second unassigned alert in the queue.

| Field | Value |
|---|---|
| When | March 27th, 2025 at 19:56 |
| Who | DMZ-MSEXCHANGE-2013 |
| Where | Windows Server 2012 R2 |
| Parent Process | C:\Users\Public\revshell.exe |
| Grandparent Process | C:\Windows\System32\inetsrv\w3wp.exe |

**Analysis:**

`w3wp.exe` is a legitimate IIS worker process that handles incoming web requests — it should never be spawning executables. In this case, it spawned `revshell.exe`, located in `C:\Users\Public\`, which is a reverse shell.

This pattern is consistent with a **web shell exploit**: an attacker uploaded or injected a web shell into the Exchange server, then used it to execute a reverse shell and gain remote code execution. The fact that this is Exchange 2013 (an outdated, unpatched version) makes this especially plausible — Exchange has a well-documented history of critical vulnerabilities.

**Five W's Report:**

**What:** A reverse shell (`revshell.exe`) was spawned by the IIS worker process (`w3wp.exe`), indicating remote code execution achieved via a web shell on an Exchange server.

**When:** March 27th, 2025 at 19:56

**Who:** DMZ-MSEXCHANGE-2013

**Where:** Windows Server 2012 R2

**Why:** `w3wp.exe` is a legitimate IIS process that handles web requests — it should never spawn executables. `revshell.exe` in `C:\Users\Public\` is a clear indicator of compromise. This pattern is consistent with a web shell being used to achieve command execution, likely via a known vulnerability in the outdated Exchange 2013 instance. This is a true positive and requires immediate escalation.

**Verdict: True Positive — Escalated to L2**  
**Flag:** `THM{looks_like_webshell_via_old_exchange}`

---

## Task 5 — SOC Communication

Key communication rules for L1 analysts during incidents:

- If L2 is unavailable during a critical threat → call L2, then L3, then your manager. Do **not** go to the manager first.
- Never communicate sensitive incident details over a channel that may be compromised
- If overwhelmed, focus on critical alerts only and inform your L2
- If you think you missed or misclassified a malicious alert → contact L2 immediately
- If the SIEM is down → do not skip alerts, investigate what you can with available tools

### Questions

**Q: Should you first try to contact your manager in case of a critical threat?**  
**A: No** — Try L2 first, then L3, then the manager.

**Q: Should you immediately contact your L2 if you think you missed an attack?**  
**A: Yes** — Always flag potential misses to L2 right away.

---

## Key Takeaways

- The Five W's framework gives your reports structure and makes them useful for L2 analysts who inherit your work
- SPF and DKIM failures are strong, objective evidence of email spoofing — always check these on phishing alerts
- Escalation isn't a failure — it's part of the process. When in doubt, escalate.
- Process names and parent/child relationships in process trees are critical IOCs — a legitimate process spawning something unexpected is always suspicious
- Running outdated software (Exchange 2013) creates significant attack surface — worth noting in your report
