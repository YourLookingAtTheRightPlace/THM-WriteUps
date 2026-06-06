Notion Link: https://www.notion.so/SOC-L1-Alert-Reporting-377792efbd4d80fa9a1deef5d8ab6158?source=copy_link

During/after triage, L1 analysts may be unsure how to classify an alert and need senior or system-owner input. They may also face real attacks requiring immediate action. This room introduces **alert reporting**, **escalation**, and **communication**.

# Task - 1 (Introduction)

- You Access the SIEM Dashboard link, to accomplish the tasks noted later on.

# Task - 2 (Alert Funnel)

- In this task we learn what happens after a L1 does his job, most of alerts are carried by the L1’s and are usually false positive’s, but the ones that require deeper analysis will be carried over by the L2’s, and in the image it shows how alerts enter and leave.

!image.png

and in this task we have two terms, 

**Alert Reporting:** like how we did in the last room, we made a concise report for every alert we close. and for every True Positive these reports are used for further analysis for the L2’s Instead of starting from zero.

**Alert Escalation:** are the Alerts that were escalated from a L1 to a L2. for being True Positive’s.

Question - What is the process of passing suspicious alerts to an L2 analyst for review?

Answer - Alert Escalation

What is the process of formally describing alert details and findings?

Answer - Alert Reporting

# Task - 3 (Reporting Guide)

now we learn how to report properly, instead of what we were doing.

the report format is based on the Five W’S

Why: reason for your final verdict

Who: the user that caused the alert

Where: which device or ip

What: What exact action or event sequence was performed

when: When exactly did the suspicious activity start and ended

and now we get to practicing:

we open link and we read the question : (According to the SOC dashboard (opens in new tab), which user email leaked the sensitive document?)

so we search for the alert with something related to senstive document and we find the  “Sensitive Document Share to External” so dropdown and find the “Source User Email” and thats the answer “m.boslan@tryhackme.thm”

then we go to the next question: Looking at the new alerts, who is the "sender" of the suspicious, likely phishing email?, we look at new alerts by clicking the Timer Icon to make it show us the newest alerts, then we search the phishing related one and dropdown and find the sender and input what we found which was “support@microsoft.com”, which is incorrect. for several reasons, that we’ll list later on. 

Question three - 

Open the phishing alert, read its details, and try to understand the activity.

Using the Five Ws template, what flag did you receive after writing a good report?

**Note:** Do not change the status yet, fill in the Analyst Comment and click Save.

what we do is now open the phishing alert and we write the report.

and write the five W’s, we start with the What
what is the exact action, its a phishing email. then we go with when: it was at Mar 27th 2025 at 19:25 then we go with the where? the email was sent to Eddie Huffman, IT Managere.huffman@tryhackme.thm, and then we go to the Who: and its the sender “Microsoft Supportsupport@microsoft.com” , and now the why. the reason for our final verdict. which is the most important W, this is a phishing email because firstly support doesn’t come from an email, and this is not the official support email its a phishing attempt and its talking about “600% price increase; urgent notice; download the report; read the details;” which is very suspicious and manipulative in terms of urgency 

and we have three more smoking guns which are SPF/Fail; DKIM/Fail;

- **SPF** (Sender Policy Framework) — checks if the sending server is authorized to send email on behalf of that domain. Fail = it isn't.
- **DKIM** (DomainKeys Identified Mail) — checks if the email was cryptographically signed by the domain. Fail = it wasn't.

and also a **REPORT.rar** — the rar attachment in a phishing email is a classic delivery method for malware.

which ended up as

**What:** Phishing Email

**When:** March 27th, 2025 at 19:25

**Who:** `support@microsoft.com` (spoofed — impersonating Microsoft Support)

**Where:** Delivered to Eddie Huffman, IT Manager `<e.huffman@tryhackme.thm>`

**Why:** Several indicators confirm this is a phishing attempt:

- **Spoofed Sender** — Microsoft does not initiate contact through a generic support email, and `support@microsoft.com` is not an official Microsoft support address
- **SPF/Fail + DKIM/Fail** — Both email authentication checks failed, meaning this email was not sent from Microsoft's servers. The sender domain is definitively spoofed
- **Social Engineering** — The email uses urgency tactics ("600% price increase", "urgent notice", "download the report") to pressure the recipient into acting without thinking
- **Suspicious Attachment** — A `REPORT.rar` file was attached, a common delivery method for malware in phishing campaigns

and we recieved the flag “THM{nice_attempt_faking_microsoft_support}”

# Task - 4 (Escalation Guide)

After I make a verdict and write my alert report, I decide whether to escalate to L2. I escalate when:

- It looks like a **true positive** or active attack
- The impact is **high** (data loss, malware, privilege abuse)
- I need **deeper investigation** or system-owner input
- I’m **not confident** in my conclusion / need a second opinion
- It affects **multiple users/hosts** or keeps repeating

all you need to escalate is to assign it to a L2 on shift. and you may need to write a formal request.

and now we go back to our reported alert, on  the last task we did, and now we do three things, we put it “In Progress” , change the Verdict to a “True Positive” and assign our active L2, E.Fleming (L2). and thus we get our flag:

THM{good_job_escalating_your_first_alert}

and now in our second question - 

Now, investigate the second new alert in the queue and provide a detailed alert comment.

Then, decide if you need to escalate this alert and move on according to the process.

After you finish your triage, you should receive a flag, which is your answer!

we go select the only other alert that isn’t closed, and we put in first the report, with the 5 w’s

when for the time, where the host’s os, who the host’s name, what: an attempted breach. and the why, we can see it at the 

Parent Process:

C:\Users\Public\revshell.exe

Grandparent Process:

C:\Windows\System32\inetsrv\w3wp.exe

they are operating at system32, and using a program called “revshell.exe” is reverse shell and its being used to try to escalate privilages, and “w3wp.exe” is a worker process, its taking information and requests from outside. 

so we’ll have this final report and this flag: THM{looks_like_webshell_via_old_exchange}

 

When:Mar 27th 2025 at 19:56
Who:DMZ-MSEXCHANGE-2013
Where:Windows Server 2012 R2
What: An attempt to breach the system.
Why:

`w3wp.exe` is a legitimate IIS process that handles web requests — it should never be spawning executables. `revshell.exe` located in `C:\Users\Public\` is a clear indicator of compromise. This pattern is consistent with a web shell being used to achieve command execution, likely via a vulnerability in the old Exchange 2013 instance.
**Task - 5 (Soc Communication)**

In this task we basically just need to understand that

- If a L2 isn’t available during an emergency we need to → try to call L2, then L3, and finally your manager.
- to not validate a user with a compromised or a breached chat
- if we get overwhelmed, the critical ones only, and inform your L2
- if you feel like you missclassified or missed a milicious action, reach out to an L2
- if there is any problems with the siem program do not skip the alerts, investiage what you can.

Should you first try to contact your manager in case of a critical threat (Yea/Nay)?

Nay

Should you immediately contact your L2 if you think you missed the attack (Yea/Nay)?

Yea

these questions are pretty self explantory.
