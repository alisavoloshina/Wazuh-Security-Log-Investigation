# Wazuh Security Log Investigation

The investigation focuses on detecting and analyzing repeated failed authentication attempts on a Windows device and determining whether the activity is consistent with normal user behavior or potentially suspicious authentication activity.

### Objective

* Detect failed Windows authentication attempts using Wazuh.
* Identify the affected user account and source IP address.
* Analyze Windows Security Event ID `4625`.
* Build a timeline of authentication events.
* Correlate failed authentication attempts with a subsequent successful login.
* Assess whether the activity is a false positive or potentially suspicious.
* Document the investigation process and recommended security actions.


# Investigation Scenario

Multiple failed authentication attempts were recorded on a Windows device.

The investigation was conducted to answer the following questions:

1. Which user account was targeted?
2. Where did the authentication attempts originate?
3. How many failed attempts occurred?
4. When did the attempts occur?
5. What type of authentication was used?
6. Was a successful login performed after the failed attempts?
7. Does the activity reflect normal user behavior or potentially suspicious activity?

The authentication failures were intentionally generated in a controlled laboratory environment.


# 1. Detection

The investigation began after a failed authentication event was detected in the Wazuh Threat Hunting dashboard.


![Wazuh detected a failed login attempt.](screenshots/01-wazuh-detected-failed-login.png)

**Figure 1 — Wazuh detection of Windows authentication attempts.**


# 2. Initial Event Analysis

After identifying the failed authentication event, the detailed event information was examined to determine the context of the activity.

The investigation focused on the following fields:

* `eventID`
* `targetUserName`
* `ipAddress`
* `logonType`
* `timestamp`
* `failureReason`
* `status`
* `subStatus`

These fields help answer the key questions of a SOC investigation:

**Who?** Which user account was targeted?

**Where?** Which source IP address generated the authentication attempt?

**When?** When did the event occur?

**What happened?** Why did Windows reject the authentication attempt?

**How?** What type of logon was attempted?


![Windows Event ID 4625 details](screenshots/02-windows-4625-event-details.png)

**Figure 2 — Detailed Windows Event ID 4625 information in Wazuh.**


# 3. Authentication Timeline

The next step was to examine multiple authentication failures rather than investigating a single event in isolation.

The events were reviewed chronologically to determine whether the failed attempts represented a recurring pattern.

Example timeline:

```text
10:24:14 — Failed authentication
10:24:18 — Failed authentication
10:24:21 — Failed authentication
10:23:44 — Failed authentication
```

The following attributes were compared across the events:

* Target username
* Source IP address
* Windows host
* Timestamp
* Event ID
* Logon type

This made it possible to analyze individual log events as a single authentication sequence.


![Failed login timeline](screenshots/03-failed-login-timeline.png)

**Figure 3 — Timeline of repeated failed authentication attempts.**


# 4. Correlation With Successful Authentication

A key part of the investigation was determining whether a successful authentication occurred after the failed attempts.

Windows Event ID `4624` was used to identify successful logons.

Example search:

```text
data.win.system.eventID:4624
```

The successful and failed authentication events were then compared.

Example sequence:

```text
4625 — Failed logon
4625 — Failed logon
4625 — Failed logon
4625 — Failed logon
4624 — Successful logon
```

The following attributes were compared across the events:

* Username
* Source IP address
* Host
* Timestamp
* Logon type

A successful authentication following several failed attempts **does not by itself prove account compromise**. However, this sequence warrants further investigation, especially if the source, timing, or authentication context is unusual.


![Failed login attempts followed by a successful login](screenshots/04-failed-followed-by-successful-login.png)

**Figure 4 — Failed authentication attempts followed by a successful authentication.**


# Investigation Findings

The investigation established the following:

* Multiple Windows authentication failure events were generated.
* Wazuh successfully collected and displayed the event information.
* The affected user account was identified from the event data.
* The source IP address was identified.
* The authentication attempts were analyzed chronologically.
* A subsequent Windows Event ID `4624` was analyzed to determine whether successful authentication occurred after the failed attempts.

Therefore, the activity was investigated as a **potentially suspicious authentication pattern** rather than automatically classified as an attack.


# False Positive Assessment

Multiple failed login attempts do not necessarily indicate malicious activity.

Possible legitimate explanations include:

* A user entered an incorrect password.
* A user forgot their password.
* An expired or recently changed password was used.
* Repeated authentication attempts originated from a legitimate workstation.

Additional indicators that would increase the level of suspicion include:

* An unknown source IP address.
* Authentication attempts outside normal working hours.
* Multiple accounts being targeted from the same source.
* A large number of authentication failures.
* Authentication attempts against privileged accounts.
* Successful authentication immediately following repeated failed attempts.
* Other suspicious activity originating from the same endpoint.

Therefore, authentication failures should be evaluated **in context and within their timeline**, rather than treated as confirmed malicious activity solely because Windows Event ID `4625` was generated.

---

# Conclusion

During the investigation, individual failed login attempts were not treated as confirmed security incidents. Instead, the events were analyzed using **user, source, timestamp, authentication type, event sequence, and subsequent successful authentication** to determine the appropriate security context.

The key takeaway from the investigation is:

> **A security alert is the starting point of an investigation, not its conclusion.**

A SOC analyst should collect sufficient evidence, establish the context, determine whether the behavior is expected, and escalate the activity when the available evidence justifies further investigation.
