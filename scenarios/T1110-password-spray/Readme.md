# T1110.003 Password Spraying

## Objective

Simulate a password spraying attack against Active Directory users, detect the activity using Wazuh SIEM, and validate defensive controls.

---

## Attack

**Tool used:**

* crackmapexec

**Command:**

```
crackmapexec smb 10.10.10.X -u users.txt -p WrongPassword123
```
![first attack](Screenshots/1_crackmapexec_attack.png)

**Description:**
A password spraying attack was performed by attempting authentication with a common password across multiple domain user accounts.

**Result:**

* Multiple failed authentication attempts were generated across domain users.
* No account lockouts occurred during this phase.

![second logs](Screenshots/2_event_logs.png)


---

## Detection

### Windows Event Logs

* **Event ID 4625** — Failed logon attempts

### Wazuh SIEM

* Rule triggered: **T1110 – Multiple Windows logon failures**
* Alert level: **High (Level 10)**
* Detection based on correlation of multiple failed logon events

![third wazuh_logs](Screenshots/3_Wazuh_detections.png)

---

## Issues Encountered

### 1. Wazuh Agents Disconnected

**Impact:**

* No log ingestion into SIEM
* Detection pipeline was non-functional

**Root Cause:**

* Incorrect Wazuh manager IP in agent configuration
* Missing/invalid agent authentication keys

**Resolution:**

* Re-registered agents using `agent-auth`
* Updated `ossec.conf` with correct Wazuh manager IP
* Restarted agent services

---

### 2. Missing Event ID 4740 (Account Lockout)

**Impact:**

* Account lockout events were not visible in logs or SIEM

**Root Cause:**

* Audit policy for **User Account Management** was not enabled

**Resolution:**

```
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
```

![fourth lockout_policy_empty](Screenshots/4_lockout_policy_empty.png)

---

## Hardening

Implemented **Account Lockout Policy** via Group Policy:

* Lockout threshold: **5 failed attempts**
* Lockout duration: **15 minutes**
* Reset counter after: **15 minutes**

![fifth lockout_policy_done](Screenshots/5_lockout_policy_done.png)
![sixth more_logs](Screenshots/6_more_event_logs.png)

---

## Validation

To validate the lockout policy, repeated authentication attempts were performed against a single user account.

**Result:**

* Account was successfully locked after exceeding threshold
* **Event ID 4740 (Account locked out)** was generated
* Event visibility required filtering due to log volume
* Detection confirmed in both Windows Event Logs and Wazuh SIEM

![seventh 4740_event](Screenshots/7_4740_event.png)
