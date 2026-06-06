# THM Write-Up: SOC L1 Alert Triage

**Platform:** TryHackMe  
**Path:** SOC Level 1  
**Difficulty:** Easy  
**Date:** 2025

---

## Overview

This room covers SOC alert triage — the process of reviewing, classifying, and resolving security alerts as an L1 analyst. A missed or misclassified alert can be the difference between catching a breach early and missing it entirely. The room walks through alert properties, prioritisation logic, and hands-on triage using a simulated SIEM dashboard.

---

## Task 1 — Introduction

Access to the SOC SIEM Dashboard is granted to complete the tasks in this room.

---

## Task 2 — Events and Alerts

As an L1 analyst, this is your reality: millions of logs, a SIEM firing alerts, and you're the first filter between noise and a real breach.

Everything that happens on a computer — logins, file downloads, programs launching — is an **event**. Agents collect these events and forward them to a centralised platform. From there, detection rules decide which events are worth flagging as **alerts**. A SOC team can receive millions of logs per day from thousands of different systems.

### Alert Management Platforms

| Solution | Programs | Description |
|---|---|---|
| SIEM System | Splunk ES, Elastic | Solid alert management — the standard choice for most SOC teams |
| EDR or NDR | MS Defender, CrowdStrike | Provide their own dashboards, but SIEM/SOAR is preferred |
| SOAR System | Splunk SOAR, Cortex SOAR | Used by larger SOC teams to aggregate alerts from multiple sources |
| ITSM System | Jira, TheHive | Some teams use dedicated ITSM tools for alert tracking |

### SOC Team Hierarchy

From bottom to top:

- **SOC L1 Analysts** — Review alerts, distinguish real threats from noise, escalate to L2 when needed
- **SOC L2 Analysts** — Receive escalated alerts and perform deeper analysis and remediation
- **SOC Engineers** — Ensure alerts contain enough information for efficient triage
- **SOC Manager** — Tracks speed and quality of triage to ensure real attacks aren't missed

### Questions

**Q: What is the number of alerts in the SOC dashboard?**  
**A: 5**

After opening the SIEM dashboard, 5 alerts are visible in the queue.

**Q: What is the name of the most recent alert?**  
**A: Double-Extension File Creation**

Sorted by time, the most recent alert in the queue is Double-Extension File Creation.

---

## Task 3 — Alert Properties

A SIEM dashboard typically shows 8 standard alert fields. As an L1, you need to quickly read these to understand what happened and decide whether to escalate.

| # | Property | Description | Examples |
|---|---|---|---|
| 1 | Alert Time | When the alert was created — usually a few minutes after the actual event | Alert: 15:35 / Event: 15:32 |
| 2 | Alert Name | Summary of what happened, based on the detection rule | Unusual Login Location, RDP Bruteforce |
| 3 | Alert Severity | Urgency level, set by detection engineers but adjustable | 🟢 Low / 🟡 Medium / 🟠 High / 🔴 Critical |
| 4 | Alert Status | Whether the alert is being worked on | 🆕 New / 🔄 In Progress / ✅ Closed |
| 5 | Alert Verdict | Whether it's a real threat or noise | 🔴 True Positive / 🟢 False Positive |
| 6 | Alert Assignee | Which analyst owns the alert and is responsible for it | — |
| 7 | Alert Description | Explains the alert: rule logic, why it's suspicious, how to triage | — |
| 8 | Alert Fields | The specific values that triggered the rule | Hostname, commandline, source IP, etc. |

### Example Scenario

An alert fires for repeated failed logins from the same location — a potential brute force. As L1, the process is:

1. Change status to **In Progress**
2. Read the alert description and review the fields
3. Investigate — in this case, the location matches the user's usual login location
4. Assign verdict: **False Positive**
5. Change status to **Closed**, set severity to Low

### Questions

**Q: What was the verdict for the "Unusual VPN Login Location" alert?**  
**A: False Positive**

**Q: What user was mentioned in the "Unusual VPN Login Location" alert?**  
**A: M.Clark**

Found by expanding the alert dropdown and checking the Source User field.

---

## Task 4 — Alert Prioritisation

With thousands of alerts in a queue, working in the right order matters. Missing a critical alert because you were triaging low-severity ones is a real failure mode.

**Priority rules:**
- Always work highest severity first (Critical → High → Medium → Low)
- Within the same severity, work oldest alerts first

### Questions

**Q: Should you prioritise medium over low severity alerts?**  
**A: Yes** — Medium is higher on the severity scale.

**Q: Should you take the newest alerts first?**  
**A: No** — Oldest alerts first within each severity level.

**Q: Assign yourself to the first-priority alert. What is its name?**  
**A: Potential Data Exfiltration**

Filtered to Critical severity, assigned to self, changed status to In Progress.

---

## Task 5 — Alert Triage

Three unassigned alerts to triage, worked highest severity first.

---

### Alert 1 — Potential Data Exfiltration (Critical)

**Description:** This rule detects 5 or more gigabytes of data sent from a single device to a single destination within a day, which may indicate data exfiltration to an untrusted location.

| Field | Value |
|---|---|
| Destination | zoom.us |
| Source IP | 192.168.45.66 |
| Source Network | UK04/MEETINGROOM |
| Sent Data | 5.8 GB |
| Received Data | 5.2 GB |

**Analysis:** The destination is zoom.us — a legitimate, trusted video conferencing platform. The source network is labelled MEETINGROOM, which strongly suggests this traffic is from an active video meeting. Large data transfer to Zoom from a meeting room is expected behaviour.

**Verdict: False Positive**  
**Comment:** Trusted URL. Data volume consistent with Zoom meeting usage.  
**Flag:** `THM{looks_like_lots_of_zoom_meetings}`

---

### Alert 2 — Double-Extension File Creation (High)

**Description:** This rule detects creation of double-extension files like `*.pdf.exe` or `*.gif.lnk`, commonly used in phishing attacks to trick users into running malicious executables.

| Field | Value |
|---|---|
| Host | LPT-HR-009 |
| Process Name | chrome.exe |
| Process User | S.Conway |
| Target File | C:\Users\S.Conway\Downloads\cats2025.mp4.exe |
| File MotW | https://freecatvideoshd.monster/cats2025.mp4.exe |
| File MD5 | 14d8486f3f63875ef93cfd240c5dc10b |

**Analysis:**

- **Double extension** — `cats2025.mp4.exe` is disguised as a video file but is actually an executable. A legitimate MP4 would never have a `.exe` extension.
- **Mark of the Web (MotW)** — Windows flags files downloaded from the internet with a MotW tag. The source URL is `freecatvideoshd.monster` — clearly not a legitimate domain.
- **VirusTotal** — The MD5 hash `14d8486f3f63875ef93cfd240c5dc10b` was submitted to VirusTotal and flagged as malicious.

**Verdict: True Positive**  
**Reasons:**
- Suspicious source URL
- Wrong file extension (double-extension masking)
- Hash confirmed malicious on VirusTotal

**Flag:** `THM{how_could_this_user_fall_for_it?}`

---

### Alert 3 — Download from Untrusted Source (Low)

**Description:** Alert triggered for a file downloaded from an external source.

**Analysis:** The source URL is `https://github.com/facebook/react` — the official Facebook React repository on GitHub. GitHub is a trusted platform and this is a legitimate open-source repository. The alert is a false positive caused by a broad detection rule that flags all external downloads.

**Verdict: False Positive**  
**Comment:** Download originated from a trusted GitHub repository. No malicious activity.  
**Flag:** `THM{should_we_allow_github_for_devs?}`

---

## Key Takeaways

- Every alert starts as an event — your job as L1 is to determine if that event is a real threat or noise
- Always work by priority: highest severity first, oldest first within the same severity
- Use all available fields: MotW, VirusTotal hash lookups, source URLs, and network context all help build your verdict
- A well-written verdict comment saves L2 analysts time if escalation is needed
