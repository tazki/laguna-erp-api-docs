# Laguna ERP Payment Transaction API Documentation

## Overview

This API allows third-party systems to submit payment transactions to Laguna ERP using JWT (JSON Web Token) authentication.

**Base_URL:** `will provide later`

---

## Authentication Flow

### Step 1: Obtain JWT Token

Third-party clients must first obtain a JWT token using their API credentials.

**Endpoint:** `POST /api/method/erpnext.payment_auth.get_payment_token`

**Headers:**

```
Content-Type: application/json
```

**Request Body:**

```json
{
  "api_key": "YOUR_API_KEY",
  "api_secret": "YOUR_API_SECRET"
}
```

**Success Response (200 OK):**

```json
{
  "status": "SUCCESS",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9",
  "token_type": "Bearer",
  "expires_in": 86400
}
```

**Error Response (401 Unauthorized):**

```json
{
  "status": "ERROR",
  "message": "Invalid API credentials",
  "errorCode": "VALIDATION_ERROR"
}
```

**cURL Example:**

```bash
curl -X POST "{Base_URL}/api/method/erpnext.payment_auth.get_payment_token" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "YOUR_API_KEY",
    "api_secret": "YOUR_API_SECRET"
  }'
```

---

## Step 2: Create Payment Transaction

Use the JWT token obtained in Step 1 to submit payment transactions.

**Endpoint:** `POST /api/method/erpnext.custom_payment_api.create_payment_transaction`

**Headers:**

```
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json
```

**Request Body:**

```json
{
  "PaymentStatusId": "SUCC",
  "EntryTypeId": "CRE",
  "CurrencyCode": "PHP",
  "TransactionGuid": "22222222-3333-4444-5555-666666666672",
  "TransactionNo": "TXN-2026-001",
  "Description": "Payment for Invoice #12345",
  "ToWalletAccountName": "Laguna Provincial",
  "Amount": 5000,
  "CreatedOn": "2026-01-21T12:00:00+08:00"
}
```

### Request Fields

| Field                 | Type   | Required | Description                                             |
| --------------------- | ------ | -------- | ------------------------------------------------------- |
| `PaymentStatusId`     | String | Yes      | Payment status code (e.g., "SUCC" for success)          |
| `EntryTypeId`         | String | Yes      | Entry type identifier (e.g., "CRE" for credit)          |
| `CurrencyCode`        | String | Yes      | ISO currency code (e.g., "PHP", "USD")                  |
| `TransactionGuid`     | String | Yes      | Unique transaction identifier (UUID format recommended) |
| `TransactionNo`       | String | Yes      | Transaction reference number                            |
| `Description`         | String | Yes      | Payment description                                     |
| `ToWalletAccountName` | String | Yes      | Destination wallet/account name                         |
| `Amount`              | Number | Yes      | Payment amount (numeric value)                          |
| `CreatedOn`           | String | Yes      | Transaction timestamp (ISO 8601 format)                 |

---

## Response Examples

### Success (200 OK)

```json
{
  "status": "SUCCESS",
  "message": "Payment received and recorded successfully.",
  "TransactionGuid": "550e8400-e29b-41d4-a716-446655440001",
  "solErpTransactionId": "SOL-2023-07-000987",
  "officialReceiptNo": "OAR-2023-07-00456",
  "fundCode": "TF",
  "collectingEntity": "Province of Laguna",
  "postingDate": "2023-07-07 09:00:00",
  "journalReference": "CRJ-2023-07-01589"
}
```

**Fields:**

- `status`: SUCCESS or ERROR
- `message`: response message
- `solErpTransactionId`: System-generated Payment Transaction ID in Laguna ERP

---

### Error: Missing Required Field (400 Bad Request)

```json
{
  "status": "ERROR",
  "message": "Invalid or incomplete payment data.",
  "errorCode": "VALIDATION_ERROR",
  "details": "Required field(s) missing: TransactionGuid"
}
```

---

### Error: Duplicate Transaction (409 Conflict)

```json
{
  "status": "ERROR",
  "message": "Duplicate transaction detected.",
  "errorCode": "DUPLICATE_TRANSACTION",
  "details": "TransactionGuid already exists in SOL ERP+."
}
```

**Note:** This error indicates the transaction was already submitted. This is an **idempotency check** - retrying with the same `TransactionGuid` will not create duplicate records.

---

### Error: Authentication Required (401 Unauthorized)

```json
{
  "status": "ERROR",
  "message": "Authentication required",
  "errorCode": "AUTHENTICATION_REQUIRED",
  "details": "Authorization header is missing or invalid."
}
```

**Common causes:**

- Missing or invalid JWT token
- Expired JWT token (tokens expire after 24 hours)
- Malformed Authorization header

---

## Complete cURL Examples

### Example 1: Get Token and Submit Payment (Success)

**Step 1 - Get Token:**

```bash
curl -X POST "{Base_URL}/api/method/erpnext.payment_auth.get_payment_token" \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "api_key_here",
    "api_secret": "api_secret_here"
  }' \
  | jq -r '.token'
```

**Step 2 - Submit Payment:**

```bash
curl -X POST "{Base_URL}/api/method/erpnext.custom_payment_api.create_payment_transaction" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "PaymentStatusId": "SUCC",
    "EntryTypeId": "CRE",
    "CurrencyCode": "PHP",
    "TransactionGuid": "550e8400-e29b-41d4-a716-446655440001",
    "TransactionNo": "TXN-2026-001",
    "Description": "Payment for Invoice #12345",
    "ToWalletAccountName": "Laguna Provincial",
    "Amount": 5000.00,
    "CreatedOn": "2026-01-21T12:00:00+08:00"
  }'
```

**Expected Response:**

```json
{
  "status": "SUCCESS",
  "message": "Payment received and recorded successfully.",
  "TransactionGuid": "550e8400-e29b-41d4-a716-446655440001",
  "solErpTransactionId": "SOL-2023-07-000987",
  "officialReceiptNo": "OAR-2023-07-00456",
  "fundCode": "TF",
  "collectingEntity": "Province of Laguna",
  "postingDate": "2023-07-07 09:00:00",
  "journalReference": "CRJ-2023-07-01589"
}
```

---

## Best Practices

### 1. Token Management

- **Cache tokens:** Tokens are valid for 24 hours. Cache and reuse them instead of requesting a new token for each transaction.
- **Token refresh:** Request a new token when you receive a 401 error.
- **Secure storage:** Store API credentials and tokens securely. Never expose them in client-side code.

### 2. Idempotency

- **Use unique GUIDs:** Always generate a unique `TransactionGuid` for each transaction (UUID v4 recommended).
- **Handle 409 errors:** A 409 response means the transaction was already processed. Check the `existing_name` field to retrieve the original transaction ID.
- **Retry logic:** On network failures, retry with the **same** `TransactionGuid` to prevent duplicate transactions.

### 3. Error Handling

```javascript
// Example error handling in JavaScript
async function submitPayment(paymentData) {
  try {
    const response = await fetch(
      "{Base_URL}/api/method/erpnext.custom_payment_api.create_payment_transaction",
      {
        method: "POST",
        headers: {
          Authorization: `Bearer ${jwtToken}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify(paymentData),
      }
    );

    const result = await response.json();

    if (response.status === 401) {
      // Token expired or invalid - get new token
      await refreshToken();
      return submitPayment(paymentData); // Retry
    }

    if (result.status === "ERROR") {
      // Optionally handle duplicate transaction
      if (result.errorCode === "DUPLICATE_TRANSACTION") {
        console.warn("Duplicate transaction detected.");
        // You may want to return a special value or handle accordingly
      }
      throw new Error(result.details || result.message || result.errorCode);
    }

    return { success: true, transactionId: result.solErpTransactionId };
  } catch (error) {
    console.error("Payment submission failed:", error);
    throw error;
  }
}
```

---

## HTTP Status Codes

| Code | Meaning      | Description                                       |
| ---- | ------------ | ------------------------------------------------- |
| 200  | OK           | Request successful                                |
| 400  | Bad Request  | Invalid request format or missing required fields |
| 401  | Unauthorized | Missing, invalid, or expired JWT token            |
| 409  | Conflict     | Duplicate transaction (idempotency check)         |
| 500  | Server Error | Internal server error                             |

## Changelog

### Version 1.0 - January 21, 2026

- Initial API release
- JWT authentication implementation
- Payment transaction creation endpoint
- Idempotency support via TransactionGuid
