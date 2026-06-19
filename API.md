# eSIM E-Go API Documentation

**Base URL:** `https://api.esim-ego.com/api/v1`

**Authentication:** Bearer JWT token via `Authorization: Bearer <access_token>` header.

**Content-Type:** `application/json` (unless noted otherwise).

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [User](#2-user)
3. [Plans](#3-plans)
4. [Orders](#4-orders)
5. [Wallet](#5-wallet)
6. [Payments](#6-payments)
7. [Push Notifications](#7-push-notifications)
8. [Admin — Users](#8-admin--users)
9. [Admin — Orders](#9-admin--orders)
10. [Admin — Payments](#10-admin--payments)
11. [Admin — Finance](#11-admin--finance)
12. [Admin — Plans](#12-admin--plans)
13. [Admin — Inventory](#13-admin--inventory)
14. [Admin — Settings](#14-admin--settings)
15. [Admin — Analytics](#15-admin--analytics)
16. [Admin — Backup](#16-admin--backup)
17. [Admin — Support](#17-admin--support)
18. [Admin — Referral](#18-admin--referral)
19. [Admin — Audit Logs](#19-admin--audit-logs)
20. [Admin — 2FA](#20-admin--2fa)
21. [Webhooks / Callbacks](#21-webhooks--callbacks)
22. [Error Codes](#22-error-codes)

---

## 1. Authentication

### `POST /auth/register`

Create a new user account.

**Request:**
```json
{
  "phone": "9647XXXXXXXXX",
  "name": "John Doe",
  "language": "en",
  "timezone": "Asia/Baghdad",
  "referral_code": "ABC123"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "phone": "9647XXXXXXXXX",
    "name": "John Doe",
    "language": "en",
    "timezone": "Asia/Baghdad"
  }
}
```

### `POST /auth/send-otp`

Send OTP verification code.

**Request:**
```json
{
  "phone": "9647XXXXXXXXX"
}
```

**Response:**
```json
{
  "success": true,
  "message": "OTP sent successfully"
}
```

### `POST /auth/verify-otp`

Verify OTP and authenticate.

**Request:**
```json
{
  "phone": "9647XXXXXXXXX",
  "code": "123456"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGci...",
    "refresh_token": "eyJhbGci...",
    "user": {
      "id": "uuid",
      "phone": "9647XXXXXXXXX",
      "name": "John Doe",
      "role": "user",
      "is_active": true,
      "language": "en"
    },
    "2fa_required": false
  }
}
```

### `POST /auth/refresh`

Refresh access token.

**Request:**
```json
{
  "refresh_token": "eyJhbGci..."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGci...",
    "refresh_token": "eyJhbGci..."
  }
}
```

### `POST /auth/logout`

Invalidate tokens.

**Request:**
```json
{
  "refresh_jti": "uuid (optional)"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## 2. User

All user endpoints require `Authorization: Bearer <token>`.

### `GET /user/profile`

Get current user profile.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "phone": "9647XXXXXXXXX",
    "name": "John Doe",
    "email": null,
    "role": "user",
    "language": "en",
    "timezone": "Asia/Baghdad",
    "is_active": true,
    "referral_code": "ABC123",
    "created_at": "2026-06-18T10:00:00+00:00"
  }
}
```

### `PUT /user/profile`

Update profile.

**Request** (partial):
```json
{
  "name": "New Name",
  "language": "ar",
  "timezone": "Asia/Baghdad"
}
```

### `DELETE /user/profile`

Delete (deactivate) account.

### `GET /user/esims`

List user's eSIMs. Query: `page` (1), `limit` (20).

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "iccid": "89012345678901234567",
        "plan_name": "Iraq 1GB / 7d",
        "inventory_status": "activated",
        "data_usage_mb": 256,
        "plan_data_amount_mb": 1024,
        "plan_duration_days": 7,
        "activated_at": "2026-06-18T10:00:00+00:00",
        "expires_at": "2026-06-25T10:00:00+00:00"
      }
    ],
    "total": 1,
    "page": 1,
    "limit": 20
  }
}
```

### `GET /user/esims/<item_id>`

Get single eSIM detail.

### `GET /user/esims/<item_id>/qr`

Download QR code image (PNG).

### `POST /user/esims/<item_id>/renew`

Renew eSIM.

### `GET /user/orders/<order_id>/invoice`

Download invoice PDF.

### `GET /user/activity`

Get user activity log.

### `POST /user/orders/<order_id>/reorder`

Reorder a previous order.

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "status": "paid",
    "total_price_iqd": 15000
  }
}
```

### `POST /user/support/tickets`

Create support ticket.

**Request:**
```json
{
  "subject": "Help needed",
  "message": "My eSIM is not working",
  "priority": "high"
}
```

### `GET /user/support/tickets`

List tickets.

### `GET /user/support/tickets/<ticket_id>`

Get ticket detail.

### `POST /user/support/tickets/<ticket_id>/messages`

Add message to ticket.

**Request:**
```json
{
  "message": "I tried restarting, still not working"
}
```

### `POST /user/support/tickets/<ticket_id>/close`

Close ticket.

### `GET /user/referral/code`

Get referral code.

### `POST /user/referral/code`

Apply referral code.

**Request:**
```json
{
  "code": "ABC123"
}
```

### `GET /user/referral/stats`

Get referral statistics.

---

## 3. Plans

Public endpoints (no auth required).

### `GET /plans`

List active plans. Query: `page` (1), `limit` (20).

**Response (Cache-Control: 300s):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "name": "Iraq 1GB / 7d",
        "description": "1GB data for 7 days in Iraq",
        "data_amount_mb": 1024,
        "duration_days": 7,
        "price_iqd": 15000,
        "price_usd": "10.00",
        "countries": "Iraq",
        "is_active": true,
        "sort_order": 1
      }
    ],
    "total": 1,
    "page": 1,
    "limit": 20
  }
}
```

### `GET /plans/<plan_id>`

Get single plan detail.

---

## 4. Orders

Requires authentication.

### `POST /orders`

Create a new order (purchase).

**Request:**
```json
{
  "plan_id": "uuid",
  "quantity": 1,
  "idempotency_key": "unique_key_8chars_min",
  "coupon_code": "DISCOUNT10"
}
```

**Response (201) — Auto mode:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "status": "paid",
    "plan_name": "Iraq 1GB / 7d",
    "quantity": 1,
    "base_price_iqd": 15000,
    "discount_amount": 1500,
    "tax_amount": 0,
    "total_price_iqd": 13500,
    "balance_before": 50000,
    "balance_after": 36500,
    "esim": {
      "id": "uuid",
      "iccid": "89012345678901234567",
      "qr_code": "base64...or LPA:..."
    }
  }
}
```

**Response (201) — Manual approval mode:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "status": "pending_approval",
    "plan_name": "Iraq 1GB / 7d",
    "quantity": 1,
    "base_price_iqd": 15000,
    "total_price_iqd": 13500
  }
}
```

### `GET /orders`

List user's orders. Query: `page`, `limit`.

### `GET /orders/<order_id>`

Get order detail.

---

## 5. Wallet

Requires authentication.

### `GET /wallet`

Get wallet balance and details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "balance": 50000,
    "frozen_balance": 0,
    "available": 50000
  }
}
```

### `GET /wallet/transactions`

List transactions. Query: `page`, `limit`.

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "amount": 13500,
        "type": "payment",
        "balance_before": 50000,
        "balance_after": 36500,
        "description": "Payment for order abc12345",
        "created_at": "2026-06-18T10:00:00+00:00"
      }
    ],
    "total": 1,
    "page": 1,
    "limit": 20
  }
}
```

### `POST /wallet/deposit`

Initiate a deposit.

**Request:**
```json
{
  "amount": 50000,
  "payment_method": "zaincash",
  "idempotency_key": "unique_key_8chars_min"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "payment_id": "uuid",
    "payment_url": "https://pg-api-uat.zaincash.iq/...",
    "expiry_time": "2026-06-18T10:10:00+00:00"
  }
}
```

---

## 6. Payments

### `GET /payments/<payment_id>`

Get payment status. Requires auth.

### `POST /payments/deposit/confirm`

Manually confirm a deposit. Requires auth.

**Request:**
```json
{
  "payment_id": "uuid",
  "provider_transaction_id": "txn_123"
}
```

### `POST /payments/zaincash/init`

Direct ZainCash initiation. Requires auth.

**Request:**
```json
{
  "amount": 50000,
  "idempotency_key": "unique_key",
  "return_url": "https://example.com/callback"
}
```

### `GET|POST /payments/zaincash/callback`

ZainCash callback endpoint (public, called by ZainCash).

**Query param:** `token` (JWT)

### `POST /payments/qicard/init`

Direct QiCard initiation. Requires auth.

**Request:**
```json
{
  "amount": 50000,
  "idempotency_key": "unique_key"
}
```

### `POST /payments/qicard/webhook`

QiCard webhook (public, called by QiCard). Header: `X-Signature`.

### `GET /payments/qicard/status/<request_id>`

Check QiCard payment status. Requires auth.

### `POST /payments/qicard/cancel`

Cancel QiCard payment. Requires auth.

**Request:**
```json
{
  "payment_id": "uuid"
}
```

---

## 7. Push Notifications

Requires authentication.

### `POST /push/register`

Register device for push notifications.

**Request:**
```json
{
  "token": "fcm_device_token_here",
  "platform": "android"
}
```

**Platform:** `"ios"` or `"android"`.

### `DELETE /push/unregister`

Unregister device token.

**Request:**
```json
{
  "token": "fcm_device_token_here"
}
```

### `GET /push/tokens`

List active push tokens.

**Response:**
```json
{
  "success": true,
  "data": {
    "tokens": ["fcm_token_1", "fcm_token_2"],
    "total": 2
  }
}
```

---

## 8. Admin — Users

All admin endpoints require `Authorization: Bearer <token>` with role `admin` or `superadmin`.

### `GET /admin/users`

List all users. Query: `page`, `limit`.

### `GET /admin/users/<user_id>`

Get user detail.

### `POST /admin/users/<user_id>/toggle-active`

Toggle user active/inactive.

### `GET /admin/users/search`

Search users. Query: `q` (search term), `role`, `is_active`, `sort_by`, `sort_order`, `page`, `limit`.

### `PUT /admin/users/<user_id>`

Update user. Body: any subset of `name`, `language`, `timezone`, `is_active`, `role`.

### `POST /admin/users/<user_id>/ban`

Ban user (set `is_active=false`).

### `POST /admin/users/<user_id>/unban`

Unban user (set `is_active=true`).

### `GET /admin/users/<user_id>/wallet`

Get user's wallet.

### `GET /admin/users/<user_id>/wallet/transactions`

Get user's wallet transactions.

### `POST /admin/users/<user_id>/wallet/adjust`

Adjust user's wallet balance.

**Request:**
```json
{
  "amount": 50000,
  "reason": "Compensation for issue"
}
```

**Note:** `amount` must be positive (`> 0`, `<= 100,000,000`).

---

## 9. Admin — Orders

### `GET /admin/orders`

List all orders. Query: `page`, `limit`, `status`, `user_id`, `plan_id`, `sort_by`, `sort_order`.

### `GET /admin/orders/<order_id>`

Get order detail with user, plan, items, pricing, refund info.

### `POST /admin/orders/<order_id>/approve`

Approve a pending approval order (manual mode).

**Response:**
```json
{
  "success": true,
  "data": {
    "order_id": "uuid",
    "status": "paid",
    "balance_before": 50000,
    "balance_after": 36500,
    "esim": {
      "id": "uuid",
      "iccid": "89012345678901234567",
      "qr_code": "base64..."
    }
  }
}
```

### `POST /admin/orders/<order_id>/cancel`

Cancel order. Refunds wallet if order was paid.

**Request:**
```json
{
  "reason": "Customer request"
}
```

### `POST /admin/orders/<order_id>/refund`

Refund an order.

**Request:**
```json
{
  "amount": 15000,
  "reason": "Service issue"
}
```

`amount` is optional (defaults to full order total). Must be `> 0` and not exceed remaining refundable amount.

### `POST /admin/orders/<order_id>/reprocess`

Reprocess a failed order (re-attempt eSIM activation).

---

## 10. Admin — Payments

### `GET /admin/payments`

List all payments. Query: `page`, `limit`, `status`, `method`, `user_id`.

### `POST /admin/payments/<payment_id>/confirm`

Manually confirm a pending payment. Credits the user's wallet.

### `POST /admin/payments/<payment_id>/refund`

Refund a payment.

**Request:**
```json
{
  "reason": "Double charge"
}
```

For deposit payments (no order), the wallet balance is deducted. For order payments, a refund is processed through `RefundService`.

---

## 11. Admin — Finance

### `GET /admin/exchange-rates`

List exchange rates.

### `POST /admin/exchange-rates`

Set exchange rate.

**Request:**
```json
{
  "base_currency": "IQD",
  "target_currency": "USD",
  "rate": 0.00067
}
```

### `GET /admin/tax-rates`

List tax rates.

### `POST /admin/tax-rates`

Create tax rate (201).

**Request:**
```json
{
  "name": "VAT",
  "percentage": 10.00,
  "description": "Value Added Tax"
}
```

### `PUT /admin/tax-rates/<tax_id>`

Update tax rate.

### `GET /admin/coupons`

List coupons.

### `POST /admin/coupons`

Create coupon (201).

**Request:**
```json
{
  "code": "SUMMER20",
  "discount_type": "percentage",
  "discount_value": 20.00,
  "max_uses": 100,
  "min_order_amount": 10000,
  "max_discount_amount": 50000,
  "applicable_plan_ids": ["uuid1", "uuid2"],
  "expires_at": "2026-12-31T23:59:59+00:00"
}
```

### `PUT /admin/coupons/<coupon_id>`

Update coupon.

### `GET /admin/refunds`

List refunds (paginated).

### `POST /admin/refunds`

Create refund.

**Request:**
```json
{
  "order_id": "uuid",
  "amount": 15000,
  "reason": "Customer returned eSIM"
}
```

### `GET /admin/freezes`

List wallet freezes (paginated).

### `POST /admin/freezes`

Freeze wallet amount.

**Request:**
```json
{
  "user_id": "uuid",
  "amount": 5000,
  "reason": "Dispute investigation"
}
```

### `POST /admin/freezes/<freeze_id>/release`

Release a freeze.

### `GET /admin/reports/financial`

Financial report. Query: `period` (`"daily"`, `"weekly"`, `"monthly"`).

### `GET /admin/reports/wallet`

Wallet dashboard (total deposits, withdrawals, pending, etc.).

---

## 12. Admin — Plans

### `GET /admin/plans`

List all plans (including inactive). Ordered by `sort_order ASC`, `created_at DESC`.

### `POST /admin/plans`

Create plan (201).

**Request:**
```json
{
  "name": "Iraq 5GB / 30d",
  "description": "5GB data for 30 days",
  "data_amount_mb": 5120,
  "duration_days": 30,
  "price_usd": 25.00,
  "price_iqd": 37500,
  "markup_percentage": 20,
  "countries": "Iraq",
  "provider_bundle_id": "esim_5GB_30D_IQ",
  "is_active": true
}
```

### `PUT /admin/plans/<plan_id>`

Update plan.

### `DELETE /admin/plans/<plan_id>`

Deactivate plan (soft delete).

### `POST /admin/plans/sync-catalogue`

Sync catalogue from eSIM provider. Generates/updates plans from provider data.

**Response:**
```json
{
  "success": true,
  "data": {
    "created": 50,
    "updated": 10,
    "skipped": 2,
    "deactivated": 5,
    "total_active": 100
  }
}
```

### `PUT /admin/plans/batch`

Batch update multiple plans.

**Request:**
```json
{
  "updates": [
    {
      "id": "uuid1",
      "price_iqd": 14000,
      "is_active": true,
      "sort_order": 1
    },
    {
      "id": "uuid2",
      "price_iqd": 28000,
      "sort_order": 2
    }
  ]
}
```

---

## 13. Admin — Inventory

### `POST /admin/inventory/import`

Import eSIMs from CSV (multipart/form-data).

**Form fields:** `file` (CSV), `plan_id` (str).

### `GET /admin/inventory/batches`

List import batches.

### `GET /admin/inventory/iccid`

List eSIM inventory. Query: `page`, `limit`, `plan_id`, `status`.

### `GET /admin/inventory/stats`

Inventory statistics. Query: `plan_id`.

### `POST /admin/inventory/retry/<inventory_id>`

Retry failed eSIM activation.

### `GET /admin/inventory/expiring`

Get expiring eSIMs. Query: `days` (default 7).

### `POST /admin/inventory/purchase`

Purchase eSIMs from provider (bulk).

**Request:**
```json
{
  "plan_id": "uuid",
  "quantity": 10
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "purchased": 10,
    "iccids": ["8901...", "8902..."]
  }
}
```

### `GET /admin/plans-stock`

View plan stock overview.

---

## 14. Admin — Settings

### `GET /admin/settings`

List all system settings.

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "key": "purchase_mode",
        "value": "hybrid",
        "description": "Inventory purchase mode"
      },
      {
        "key": "order_approval_mode",
        "value": "auto",
        "description": "Order approval mode"
      }
    ]
  }
}
```

### `GET /admin/settings/<key>`

Get single setting.

### `PUT /admin/settings/<key>`

Set/update setting.

**Request:**
```json
{
  "value": "manual",
  "description": "Order approval mode"
}
```

### `DELETE /admin/settings/<key>`

Delete setting.

---

## 15. Admin — Analytics

### `GET /admin/analytics/dashboard`

Main dashboard stats.

### `GET /admin/analytics/charts/sales`

Sales chart data. Query: `days` (default 30).

### `GET /admin/analytics/charts/plans`

Plan distribution chart.

### `GET /admin/analytics/charts/users`

User growth chart. Query: `period` (`"daily"`, `"weekly"`, `"monthly"`).

### `GET /admin/analytics/activity`

Recent admin activity.

### `GET /admin/analytics/reports/<report_type>/export`

Export report as file. Query: `format` (`"csv"`).

---

## 16. Admin — Backup

### `POST /admin/backup/create`

Create a manual backup.

### `GET /admin/backup/list`

List backups.

### `GET /admin/backup/<backup_id>`

Get backup detail.

### `GET /admin/backup/<backup_id>/download`

Download backup file.

### `DELETE /admin/backup/<backup_id>`

Delete a backup.

### `GET /admin/backup/settings`

View backup settings and filesystem info.

### `PUT /admin/backup/settings/<key>`

Update backup setting.

**Request:**
```json
{
  "value": "60"
}
```

### `POST /admin/backup/cleanup`

Clean up old backups.

**Response:**
```json
{
  "success": true,
  "data": {
    "deleted": 5
  }
}
```

---

## 17. Admin — Support

### `GET /admin/support/tickets`

List all support tickets. Query: `page`, `limit`, `status`, `priority`.

### `GET /admin/support/tickets/<ticket_id>`

Get ticket detail.

### `POST /admin/support/tickets/<ticket_id>/reply`

Reply to ticket.

**Request:**
```json
{
  "message": "We have resolved your issue. Please check your eSIM."
}
```

### `PATCH /admin/support/tickets/<ticket_id>/status`

Update ticket status.

**Request:**
```json
{
  "status": "resolved"
}
```

### `POST /admin/support/tickets/<ticket_id>/assign`

Assign ticket to admin.

**Request:**
```json
{
  "assigned_to_id": "uuid (optional, defaults to current admin)"
}
```

---

## 18. Admin — Referral

### `GET /admin/referral/settings`

Get referral program settings.

### `PUT /admin/referral/settings/<key>`

Update referral setting.

**Request:**
```json
{
  "value": "5000"
}
```

### `GET /admin/referral/rewards`

List referral rewards. Query: `page`, `limit`, `status`.

### `GET /admin/referral/stats`

Referral statistics.

### `POST /admin/referral/rewards/<reward_id>/credit`

Manually credit a reward.

### `POST /admin/referral/rewards/<reward_id>/cancel`

Cancel a referral reward.

**Request:**
```json
{
  "reason": "Fraudulent referral detected"
}
```

---

## 19. Admin — Audit Logs

### `GET /admin/audit-logs`

List audit logs. Query: `page`, `limit`, `user_id`, `action`, `resource_type`.

---

## 20. Admin — 2FA

Two-factor authentication for admin accounts. These endpoints use `require_totp=False` so they work before TOTP is enabled.

### `POST /admin/2fa/setup`

Generate TOTP secret.

**Response:**
```json
{
  "success": true,
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "provisioning_uri": "otpauth://totp/...",
    "issuer": "eSIM E-Go"
  }
}
```

### `POST /admin/2fa/enable`

Enable 2FA with verification code.

**Request:**
```json
{
  "code": "123456"
}
```

### `POST /admin/2fa/disable`

Disable 2FA.

**Request:**
```json
{
  "code": "123456"
}
```

### `POST /admin/2fa/verify`

Verify 2FA during login (returns new tokens with `totp_verified` claim).

**Request:**
```json
{
  "code": "123456"
}
```

---

## 21. Webhooks / Callbacks

### `POST /esim/callback`

eSIM provider callback. Validates HMAC/API key from headers.

Handles alert types: `DataUsage`, `Topup`, `FirstAttachment`, `CountryChange`, `FirstUse`, `LowBalance`, `MSISDNEnabled`, `MSISDNDisabled`, etc.

### `POST /callback/otpiq/delivery`

OTPIQ SMS delivery report callback. Validates webhook secret + IP whitelist.

---

## 22. Error Codes

All API errors return:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": {
      "en": "English message",
      "ar": "الرسالة بالعربية",
      "ku": "پەیام بە کوردی",
      "fr": "...",
      "tr": "...",
      "fa": "...",
      "ur": "...",
      "de": "...",
      "es": "...",
      "ru": "...",
      "zh": "...",
      "pt": "..."
    },
    "status": 400
  }
}
```

### 2xx — Success

| Code | HTTP | Description |
|------|------|-------------|
| `SUCCESS` | 200 | Generic success |
| `CREATED` | 201 | Resource created |

### 4xx — Client Errors

| Code | HTTP | Description |
|------|------|-------------|
| `VALIDATION_MISSING_FIELD` | 400 | Required field missing |
| `VALIDATION_INVALID_AMOUNT` | 400 | Amount out of valid range |
| `VALIDATION_INVALID_PHONE` | 400 | Invalid phone format |
| `VALIDATION_INVALID_UUID` | 400 | Invalid UUID format |
| `VALIDATION_INVALID_ENUM` | 400 | Invalid enum value |
| `VALIDATION_INVALID_PARAMETER` | 400 | Invalid parameter |
| `VALIDATION_IDEMPOTENCY_REUSE` | 400 | Idempotency key already used |
| `AUTH_INVALID_TOKEN` | 401 | Missing/invalid JWT |
| `AUTH_EXPIRED_TOKEN` | 401 | Token expired |
| `AUTH_INVALID_REFRESH` | 401 | Invalid refresh token |
| `AUTH_INSUFFICIENT_PERMISSIONS` | 403 | Role not authorized |
| `AUTH_2FA_REQUIRED` | 403 | 2FA verification needed |
| `USER_NOT_FOUND` | 404 | User not found |
| `NOT_FOUND` | 404 | Generic not found |
| `PLAN_NOT_FOUND` | 404 | Plan not found |
| `PLAN_INACTIVE` | 400 | Plan is not active |
| `ORDER_NOT_FOUND` | 404 | Order not found |
| `ORDER_INVALID_STATUS` | 400 | Order cannot be processed in current status |
| `WALLET_NOT_FOUND` | 404 | Wallet not found |
| `WALLET_INSUFFICIENT_BALANCE` | 402 | Insufficient wallet balance |
| `WALLET_INSUFFICIENT_AVAILABLE` | 402 | Insufficient available balance (balance - frozen) |
| `PAYMENT_DUPLICATE` | 400 | Payment already processed |
| `PAYMENT_INVALID_SIGNATURE` | 400 | Invalid payment signature |
| `PAYMENT_METHOD_UNSUPPORTED` | 400 | Unsupported payment method |
| `REFUND_EXCEEDS_ORDER` | 400 | Refund exceeds order total |
| `REFUND_ORDER_NOT_PAID` | 400 | Order not in paid status |
| `COUPON_NOT_FOUND` | 404 | Coupon not found |
| `COUPON_EXPIRED` | 400 | Coupon has expired |
| `COUPON_EXHAUSTED` | 400 | Coupon usage limit reached |
| `INVENTORY_INSUFFICIENT_STOCK` | 400 | No available eSIM in inventory |
| `PROVIDER_UNAVAILABLE` | 503 | Payment/eSIM provider unavailable |
| `ZAINCASH_CALLBACK_INVALID` | 400 | Invalid ZainCash callback |
| `QICARD_WEBHOOK_INVALID` | 400 | Invalid QiCard webhook |
| `QICARD_VERIFICATION_FAILED` | 400 | QiCard amount/signature mismatch |
| `OTPIQ_INVALID_WEBHOOK` | 400 | Invalid OTPIQ webhook |

### 5xx — Server Errors

| Code | HTTP | Description |
|------|------|-------------|
| `INTERNAL_ERROR` | 500 | Unexpected server error |
| `DATABASE_ERROR` | 500 | Database operation failed |
| `PROVIDER_ERROR` | 502 | Upstream provider returned error |

---

## Response Format

### Success
```json
{
  "success": true,
  "data": { ... }
}
```

### Error
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": {
      "en": "...",
      "ar": "..."
    },
    "status": 400
  }
}
```

### Pagination
```json
{
  "success": true,
  "data": {
    "items": [ ... ],
    "total": 100,
    "page": 1,
    "limit": 20
  }
}
```

---

## Rate Limiting

| Scope | Limit |
|-------|-------|
| Global API | 30 requests per minute |
| Auth endpoints | 5 requests per minute |
| Admin endpoints | 60 requests per minute |

Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.

---

## WebSocket

Connect via Socket.IO at `/socket.io/` with JWT auth.

**Auth header:** `Authorization: Bearer <access_token>`

**Events:**

| Event | Direction | Description |
|-------|-----------|-------------|
| `connect` | Client → Server | Authenticate via JWT |
| `order_update` | Server → Client | Order status changed |
| `wallet_update` | Server → Client | Wallet balance changed |
| `push_received` | Server → Client | Push notification received (WebSocket mirror) |

Users automatically join room `user:<id>` on connect and receive events scoped to their account.
