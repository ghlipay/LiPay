# LiPayKripto API Integration

This documentation provides detailed steps for integrating both payment and withdrawal operations with the LiPayKripto API system.

## Overview

The LiPayKripto external-payment-request API provides a secure payment gateway that allows your users to make payments with cryptocurrencies (TRX, USDT, ETH).

**IMPORTANT:**
- A new JWT token must be obtained for each new payment request. Tokens are valid for a limited time.
- All API requests must be made using the **POST method ONLY**. GET, PUT, or other HTTP methods are not supported.

## API Endpoints

1. **Token Request:** `https://lipaykripto.com/api/auth/token`
2. **Payment Request:** `https://lipaykripto.com/api/external-payment-request`
3. **Withdrawal Request:** `https://lipaykripto.com/api/withdraw`

## 1. Token Request Process

### Request

**HTTP Method:** **POST** (❗ Only POST method is supported)

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "clientId": "YOUR_CLIENT_ID",
  "clientSecret": "YOUR_CLIENT_SECRET"
}
```

### Response

**Successful response:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600  // Token validity period (seconds)
}
```

**Error response:**

```json
{
  "error": "Invalid credentials"
}
```

## 2. Creating a Payment Request

### Request

**HTTP Method:** **POST** (❗ Only POST method is supported)

**Headers:**
```
Content-Type: application/json
Accept: application/json
Authorization: Bearer [RECEIVED_TOKEN]
```

**Body:**
```json
{
  "tryAmount": 100.00,                  // Amount in TRY (Turkish Lira)
  "paymentId": "ORDER_123",             // Unique order number
  "webhookUrl": "https://example.com/webhook"  // Webhook URL (required)
}
```

### Response

**Successful response:**

```json
{
  "success": true,
  "paymentUrl": "https://lipaykripto.com/payment/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "paymentId": "ORDER_123",
  "clientId": "YOUR_CLIENT_ID",
  "paymentRequestId": 505,
  "status": "pending_coin_selection"
}
```

**Error response:**

```json
{
  "success": false,
  "error": "Invalid amount or paymentId"
}
```

## 3. User Experience

1. User initiates a payment in your application
2. Your system obtains a **new token for each payment request** from LiPayKripto
3. Creates a payment request with the token
4. Redirects the user to the `paymentUrl`
5. User selects the cryptocurrency type and makes the payment
6. When the payment is completed, LiPayKripto sends a notification to your webhook URL
7. User is redirected to your post-payment page

## 4. Webhook Notifications

When the payment status changes, a **POST** request is sent to your webhook URL in the following format:

```json
{
  "try_amount": 100.00,
  "client_id": "YOUR_CLIENT_ID",
  "payment_id": "ORDER_123",
  "status": "confirmed"  // or "failed"
}
```

When you receive a webhook notification, you should return an HTTP 200 response.

## 5. Security Recommendations

1. Never use the Client Secret on the client side (front-end)
2. Use a unique `paymentId` for each payment
3. Your webhook URL should use HTTPS and be externally accessible
4. Check the payment_id to prevent duplicate processing of the same payment
5. To verify the integrity of webhook notifications, check that the client_id in the notification matches your client_id

## 6. Testing

To test your integration, you can follow these steps:

1. Request test account credentials from [support@lipaykripto.com](mailto:support@lipaykripto.com)
2. Test payment requests with small amounts in the test environment
3. Verify that your webhook URL receives notifications properly
4. Test successful payment, failed payment, and timeout scenarios

## 7. Error Codes and Explanations

| HTTP Code | Error Message                    | Description                                       |
|-----------|----------------------------------|---------------------------------------------------|
| 400       | Invalid amount                   | TRY amount must be greater than zero              |
| 400       | Invalid paymentId                | paymentId must be unique and not empty            |
| 400       | Webhook URL required             | Webhook URL is a required field                   |
| 401       | Invalid token                    | Token is invalid or expired                       |
| 401       | Invalid credentials              | Client ID or Client Secret is incorrect           |
| 429       | Too many requests                | Request limit exceeded, try again later           |
| 500       | Server error                     | Server error, try again later                     |

## 8. Creating a Withdrawal Request

You can use this endpoint to process cryptocurrency withdrawals (TRX, USDT, ETH) for your users.

### Request

**HTTP Method:** **POST** (❗ Only POST method is supported)

**Headers:**
```
Content-Type: application/json
Accept: application/json
```

**Body:**
```json
{
  "try_amount": 100.00,                  // Amount in TRY (Turkish Lira)
  "clientId": "YOUR_CLIENT_ID",          // Client ID
  "requestId": "WITHDRAW_123",           // Unique withdrawal reference
  "wallet_address": "TXBCD12345...",     // User's cryptocurrency wallet address
  "coin_type": "TRX",                    // Cryptocurrency type (TRX, USDT, or ETH)
  "signature": "f58a92bcde...",          // HMAC signature
  "webhook_url": "https://example.com/webhook",  // Webhook URL (required)
  "created_at": "2025-04-20T14:30:00Z"   // Transaction creation timestamp (ISO 8601 format)
}
```

**Creating the HMAC Signature:**

The signature is created by concatenating the following fields in the specified order:
```
try_amount + clientId + requestId + wallet_address + coin_type + webhook_url + created_at
```

For example:
```
100.00test12345WITHDRAW_123TXBCD12345...TRXhttps://example.com/webhook2025-04-20T14:30:00Z
```

This concatenated string is signed using the HMAC-SHA256 algorithm and your HMAC Secret.

### Response

**Successful response:**

```json
{
  "success": true,
  "data": {
    "withdrawId": 123,
    "status": "pending"
  }
}
```

**Error response:**

```json
{
  "success": false,
  "error": "Invalid signature",
  "details": [...]
}
```

### Withdrawal Webhook Notifications

When the withdrawal status changes, a **POST** request is sent to your webhook URL in the following format:

```json
{
  "success": true,
  "clientId": "YOUR_CLIENT_ID",
  "status": "confirmed",  // or "failed", "manual_process"
  "tryAmount": 100.00,
  "requestId": "WITHDRAW_123"
}
```

## 9. Contact and Support

If you encounter any issues during the integration process, please contact us at [support@lipaykripto.com](mailto:support@lipaykripto.com).