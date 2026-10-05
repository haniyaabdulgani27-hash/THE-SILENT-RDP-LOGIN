# THE-SILENT-RDP-LOG# OPERATION: THE SILENT RDP LOGIN

 SOC Investigation Report

 Investigation Theme

**Legitimate RDP Login, Suspicious Post-Authentication Activity**

---

## 1. Scenario

HR employee **Ananya** is working from home and connects to her office workstation, **HR-WS-017**, using the company's approved Remote Desktop service.

The RDP authentication is successful and initially appears legitimate.

A short time later, however, the SOC receives additional telemetry from the same workstation:

* A privileged session is created.
* An unusual command process starts.
* PowerShell executes unexpectedly.
* The workstation communicates with an internal server that the user does not normally access.

The SOC therefore investigates a different question:

> **If the RDP login was legitimate, why did suspicious activity begin after the login?**

This investigation focuses on **post-authentication behaviour** rather than treating the successful RDP login itself as the compromise.

---

## 2. Initial Assessment

The successful RDP event is:

**Windows Security Event 4624 — Logon Type 10**

This confirms that an RDP/Remote Interactive session was established.

At this stage, the SOC does **not** immediately classify the login as malicious.

The user's remote access is legitimate, so the investigation moves to what happened **inside the session**.

---

## 3. Investigation

### Privileged Session

Shortly after the RDP login, Windows records:

**Event 4672 — Special Privileges Assigned to New Logon**

The event shows that the session received elevated Windows privileges.

They are legitimate Windows security privileges that can also be abused.

The important question is therefore not:

> "Is this privilege malicious?"

It is:

> **"Was this level of privilege expected for this user and this session?"**

In this scenario, the privilege level is unusual for a normal HR workstation session.

This becomes the first indication that the investigation should continue.

---

## 4. Process Activity

The SOC then reviews process creation events:

**4688 / Sysmon Event 1**

The process sequence shows:

**explorer.exe → cmd.exe → powershell.exe**

The PowerShell process was not part of the user's normal HR workflow.

This is more significant than the RDP login itself because the authentication was expected, while the process behaviour was not.

---

## 5. PowerShell Activity

PowerShell Script Block Logging (**Event 4104**) provides additional context.

The activity shows commands being executed to:

* identify the current user,
* gather information about the Windows environment,
* identify internal systems,
* and access an internal resource.

None of these commands individually proves compromise.

However, their timing is important:

**Legitimate RDP → Privileged session → Command execution → PowerShell → System discovery**

The behaviour is now inconsistent with the expected activity of an HR user.

---

## 6. Network Investigation

The SOC checks Sysmon Event 3 for network connections generated after the PowerShell activity.

The workstation communicates with an internal server that the user does not normally access.

This creates another correlation:

**Authentication + Privilege + Process + PowerShell + Network**

At this point, the investigation is no longer about whether the employee was allowed to use RDP.

It is about whether something on the workstation was using the legitimate session.

---

## 7. What Changed the Investigation?

The successful RDP login was initially considered legitimate.

The suspicion increased because of the **behaviour after authentication**.

### Investigation chain

**Legitimate RDP Login**

↓

**Unexpected Privileged Session**

↓

**Unusual Process Creation**

↓

**PowerShell Execution**

↓

**System Discovery**

↓

**Unexpected Internal Connection**

This sequence is more valuable than any individual event because it provides behavioural context.

---

## 8. Primary Hypothesis

### H4 — Compromised Endpoint / Malicious Post-Authentication Activity

The working hypothesis is:

> **The employee legitimately authenticated through RDP, but the workstation or an existing process on the endpoint may already have been compromised and used the authenticated session to perform suspicious activity.**

This avoids assuming that the employee's credentials were stolen.

The investigation would continue with endpoint analysis to determine the original source of the activity.

---

## 9. Detection Idea

The detection should not simply alert on:

> **"Successful RDP login."**

That would generate many legitimate alerts.

Instead, the SOC can correlate:

```text
4624 Type 10
       ↓
Unusual privilege / 4672
       ↓
New process / 4688 / Sysmon 1
       ↓
PowerShell / 4104
       ↓
Unexpected network connection / Sysmon 3
```

If these events occur within a short period, the session can be assigned a **higher risk score**.

### Detection Concept

**Successful RDP + unusual post-login behaviour = investigate**

This is the main detection idea of the project.

---

## 10. SOC Response

Because the endpoint may already be compromised, the SOC should:

* isolate HR-WS-017 if malicious activity is confirmed,
* preserve relevant logs and evidence,
* investigate the suspicious process,
* review PowerShell activity,
* check recent network connections,
* investigate the endpoint for persistence or malware,
* review the same behaviour on other systems.

The employee's account should not automatically be treated as compromised simply because suspicious activity occurred during her legitimate session.

---

## 11. Final Assessment

### Verdict

**SUSPICIOUS POST-AUTHENTICATION ACTIVITY**

The RDP authentication itself appears legitimate.

The concern comes from the sequence of activity that follows:

> **Legitimate Login → Privileged Activity → Suspicious Process → PowerShell → Internal Network Activity**

This suggests that the **endpoint or an existing process may have been compromised**, and further forensic investigation is required to identify the root cause.

---

## 12. SOC Takeaway

A successful authentication does not automatically mean that everything happening inside the session is legitimate.

A SOC analyst should investigate:

> **Who logged in?**
> **From where?**
> **What happened after the login?**
> **Which processes ran?**
> **What did they access?**
> **Where did the system communicate?**

### Final Principle

> **“The login can be legitimate while the activity inside the session is not.”**

That is the key idea behind this investigation.
