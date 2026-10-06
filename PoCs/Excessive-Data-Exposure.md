# Excessive Data Exposure — Community Posts API

## Finding

**Title:** Excessive Data Exposure of User and Vehicle Identifiers

**Severity:** Medium

**Category:** API Security / Information Disclosure

**CWE:** CWE-200 — Exposure of Sensitive Information to an Unauthorized Actor

**OWASP API Security:** API3 — Broken Object Property Level Authorization

---

## Description

The Community Posts API exposes unnecessary user and vehicle-related information in its response.

The affected endpoint is:

```http
GET /community/api/v2/community/posts/recent?limit=30&offset=0
```

An authenticated low-privileged user can request recent community posts and receive additional properties including:

* **User ID**
* **Vehicle ID**
* **Email address**

These properties are not required to display the community-post content and unnecessarily expose application and user information to the API consumer.

The issue represents **Excessive Data Exposure** because the server returns more data than is required for the intended functionality.

---

## Affected Endpoint

```http
GET /community/api/v2/community/posts/recent?limit=30&offset=0
Host: 127.0.0.1:30080
```

**Method:** `GET`

**Authentication:** Required

**Affected Component:** Community API

---

## Prerequisites

A valid authenticated user account with normal user privileges.

Example:

```text
Account: test-a@mail.com
Role: ROLE_USER
```

---

# Proof of Concept

## Step 1 — Send an authenticated request

The following request was sent using a valid JWT:

```http
GET /community/api/v2/community/posts/recent?limit=30&offset=0 HTTP/1.1
Host: 127.0.0.1:30080
Authorization: Bearer <VALID_JWT>
Content-Type: application/json
Accept: */*
Connection: close
```

Equivalent command:

```bash
curl -s \
  'http://127.0.0.1:30080/community/api/v2/community/posts/recent?limit=30&offset=0' \
  -H 'Authorization: Bearer <VALID_JWT>' \
  -H 'Content-Type: application/json' \
  -H 'Accept: */*'
```

---

## Step 2 — Inspect the response

The API returns community-post information together with user and vehicle-related properties.

Example:

```json
{
  "posts": [
    {
      "id": 1,
      "title": "Example Post",
      "content": "Example content",
      "user_id": 14,
      "vehicle_id": 27,
      "email": "test-b@mail.com"
    }
  ],
  "total": 3
}
```

> The values above are representative. The actual response observed during testing should be included as evidence.

---

## Step 3 — Identify unnecessary data

The following properties were identified as unnecessary for the community-post listing functionality:

| Property     | Classification         | Reason                                                  |
| ------------ | ---------------------- | ------------------------------------------------------- |
| `user_id`    | Application identifier | Identifies the underlying user account                  |
| `vehicle_id` | Application identifier | Identifies an associated vehicle object                 |
| `email`      | Personal information   | Identifies/contact information associated with the user |

The community-post endpoint should return only information required to render the intended community functionality.

The exposure of these properties provides information that can potentially be used for:

* User enumeration
* Object enumeration
* Data correlation
* Further authorization testing
* BOLA/IDOR attack chaining
* Profiling of application relationships

---

# Security Impact

An authenticated attacker can retrieve user and vehicle identifiers associated with community content without requiring those identifiers for the intended operation.

The exposed information can increase the attack surface of the application by providing object identifiers that may be useful in subsequent attacks.

For example:

```text
Community Post
      │
      ├── user_id
      │
      ├── vehicle_id
      │
      └── email
           │
           ▼
   Additional API endpoints
           │
           ▼
 Potential enumeration /
 authorization testing
```

The issue becomes particularly significant if these identifiers can subsequently be supplied to other API endpoints that lack proper object-level authorization.

---

# Root Cause

The likely root cause is insufficient server-side filtering of API response properties.

The backend appears to expose properties associated with the underlying user/post/vehicle objects instead of returning a purpose-built response containing only the fields required by the community-post consumer.

A safer implementation would explicitly define the response schema:

```python
{
    "id": post.id,
    "title": post.title,
    "content": post.content
}
```

rather than serializing additional internal object properties.

---

# Recommended Remediation

### 1. Implement explicit response schemas

Return only properties required by the endpoint's intended functionality.

### 2. Remove unnecessary identifiers

Do not expose internal `user_id` or `vehicle_id` values unless they are explicitly required by the API consumer.

### 3. Protect personal information

Do not expose email addresses in community-post responses unless there is a documented business requirement.

### 4. Apply property-level authorization

Where a property is required, verify that the requesting user is authorized to receive it.

### 5. Avoid direct database-object serialization

Database models should not be returned directly from API endpoints.

Instead, map database objects to dedicated API response models.

### 6. Review related endpoints

Because `user_id` and `vehicle_id` are exposed, related endpoints accepting these identifiers should also be tested for:

* BOLA / IDOR
* Horizontal privilege escalation
* Unauthorized object access
* Data enumeration

---

# Retest

After remediation, repeat:

```http
GET /community/api/v2/community/posts/recent?limit=30&offset=0
```

with a normal `ROLE_USER` account.

### Expected result

The response should contain only information required by the community-post functionality.

For example:

```json
{
  "posts": [
    {
      "id": 1,
      "title": "Example Post",
      "content": "Example content"
    }
  ],
  "total": 3
}
```

The following properties should no longer be unnecessarily exposed:

```text
user_id
vehicle_id
email
```

---

# Evidence Summary

| Item                                       | Result                                     |
| ------------------------------------------ | ------------------------------------------ |
| Authentication required                    | Yes                                        |
| Privilege level                            | Normal user                                |
| Endpoint                                   | `/community/api/v2/community/posts/recent` |
| User ID exposed                            | Yes                                        |
| Vehicle ID exposed                         | Yes                                        |
| Email exposed                              | Yes                                        |
| Client functionality requires these fields | No                                         |
| Information disclosure demonstrated        | Yes                                        |

---

# Conclusion

The Community Posts API exposes unnecessary user and vehicle-related information, including **user IDs, vehicle IDs, and email addresses**, to authenticated users.

Although authentication is enforced, the API does not sufficiently minimize the properties returned to the client.

The issue should be remediated through **explicit response schemas, data minimization, and property-level authorization**.

Additionally, the exposed `user_id` and `vehicle_id` values should be treated as potential attack primitives and used to assess related endpoints for **BOLA/IDOR and object enumeration vulnerabilities**.
