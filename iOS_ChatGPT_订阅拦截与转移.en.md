# iOS ChatGPT Subscription Interception and Transfer Principles

> **⚠️ Disclaimer**: This tutorial is for learning and research purposes only. Please comply with applicable laws, regulations, and terms of service.

---

## Table of contents

1. [Core principles overview](#1-core-principles-overview)
2. [Interception: block callback requests](#2-interception-block-callback-requests)
3. [Transfer: change the User ID to activate on another account](#3-transfer-change-the-user-id-to-activate-on-another-account)
4. [Key requests in detail](#4-key-requests-in-detail)
5. [Response data structure](#5-response-data-structure)

---

## 1. Core principles overview

The whole flow involves two key operations: **interception** and **transfer**. Understanding these two concepts is a prerequisite for a successful operation.

### 1.1 Subscription payment flow

When a user subscribes to a ChatGPT membership on iOS, the full payment path is as follows:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Subscription payment flow                   │
│                                                                 │
│  iPhone ──▶ App Store purchase ──▶ Apple returns a payment      │
│                                    credential (fetch_token)     │
│                                       │                         │
│                                       ▼                         │
│              ChatGPT App sends the credential to RevenueCat     │
│              ──▶ activate the subscription                      │
│              (carries app_user_id to bind to a specific account)│
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Key roles

| Role | Description |
|------|------|
| **App Store** | Handles the real payment and returns an Apple-issued payment credential |
| **RevenueCat** | The third-party subscription management platform used by ChatGPT; verifies the credential and activates the subscription |
| **fetch_token** | Apple payment credential (Apple Signed Transaction), proving that payment has been completed |
| **app_user_id** | The ChatGPT account's Account ID, which determines which account the subscription is bound to |

---

## 2. Interception: block callback requests

### 2.1 Why intercept

After Apple payment completes, the ChatGPT App will **automatically** send the payment credential to RevenueCat and bind the subscription to the currently logged-in ChatGPT account. If we want to transfer the subscription to another account, we must **block this automatic callback process**.

### 2.2 URLs that need to be blocked

In Reqable, set **gateway blocking** (intercept / block) and block the following two URLs:

| No. | URL to block | Reason for blocking |
|:---:|------------|----------|
| 1 | `https://api.revenuecat.com/v1/receipts` | Prevent the Apple payment credential from being automatically sent back to the RevenueCat server |
| 2 | `https://ios.chat.openai.com/backend-api/payments/rc/ios/verify/v4-2023-04-27` | Prevent the ChatGPT backend from automatically verifying subscription status |

> **⚠️ Important**: These two URLs are blocked to **prevent the recharge callback from completing automatically**. If they are not blocked, the payment credential will be automatically bound to the currently logged-in ChatGPT account, and the later transfer operation will no longer be possible.

### 2.3 Interception sequence diagram

```
Normal flow (no interception):
iPhone payment ──▶ credential sent automatically ──▶ RevenueCat ──▶ bound to current account ✅ (cannot transfer)

Interception flow:
iPhone payment ──▶ credential sent ──✖ blocked by Reqable ──▶ credential does not reach RevenueCat
                                │
                                ▼
                    Manually capture the credential, modify it, and resend ──▶ transfer to the target account ✅
```

---

## 3. Transfer: change the User ID to activate on another account

### 3.1 Transfer principle

After Apple payment completes, the ChatGPT App sends a credential callback request to RevenueCat:

```
POST https://api.revenuecat.com/v1/receipts
```

This request body contains two core fields:

| Field | Meaning | Role |
|------|------|------|
| `fetch_token` | Apple payment credential (Apple Receipt) | Proves that the user has completed a real payment through the App Store; it is the "key" that makes the subscription take effect |
| `app_user_id` | ChatGPT account's Account ID | Determines **which ChatGPT account** this subscription is bound to |

> **💡 Key insight**: `fetch_token` is bound to the Apple ID (proving who paid), but is unrelated to the ChatGPT account. RevenueCat decides which user receives the subscription benefits **based only on `app_user_id`**.

### 3.2 Transfer steps

1. Use Reqable to capture the complete credential callback request (including a valid `fetch_token`)
2. Replace `app_user_id` in the request body with the **target account**'s Account ID
3. Resend the request to `https://api.revenuecat.com/v1/receipts`
4. After RevenueCat receives the valid credential, it activates the subscription on the target account

### 3.3 Transfer flow diagram

```
Original request: fetch_token (payment credential) + app_user_id = "Account A's ID"
                                        │
                                  change app_user_id
                                        │
                                        ▼
Transfer request: fetch_token (payment credential) + app_user_id = "Account B's ID"
                                        │
                                        ▼
                            Account B obtains the Pro 20x subscription ✅
```

### 3.4 How to get the target account's app_user_id

The target ChatGPT account's `app_user_id` can be obtained as follows:

1. While the target account is logged into the ChatGPT App
2. Use Reqable to capture any request sent to RevenueCat
3. Find the `app_user_id` field in the request body or URL

---

## 4. Key requests in detail

### 4.1 Apple payment credential callback request

```
POST https://api.revenuecat.com/v1/receipts
```

**Key headers:**

| Header | Value | Description |
|--------|----|------|
| `authorization` | `Bearer appl_rQLChslWRSKCUPBLPJtHCGjvujc` | RevenueCat API key |
| `x-client-bundle-id` | `com.openai.chat` | ChatGPT Bundle ID |
| `x-platform` | `iOS` | Platform identifier |
| `x-storekit2-enabled` | `true` | Using StoreKit 2 |

**Key body fields:**

| Field | Value | Description |
|------|----|------|
| `fetch_token` | `eyJhbGci...` (JWS format) | Apple-issued payment credential; **the core of the transfer** |
| `app_user_id` | `46ee5a3b-97d3-4da0-baa8-061ecf9f1b25` | ChatGPT account ID; **this field must be replaced during transfer** |
| `product_id` | `oai_chatgpt_go_1000_1m` | Subscription product ID |
| `price` | `8` | Price (USD) |
| `store_country` | `USA` | Store region |
| `transaction_id` | `440003340653016` | App Store transaction ID |

### 4.2 Fields that need to be changed during transfer

| Field | Action | Description |
|------|------|------|
| `app_user_id` | ✏️ **Must change** | Replace with the target account's Account ID |
| `fetch_token` | ❌ Keep unchanged | This is a valid payment credential issued by Apple |
| `app_transaction` | ❌ Keep unchanged | Apple's app transaction credential |
| Other fields | ❌ Keep unchanged | Including all parameters in the headers |

---

## 5. Response data structure

### 5.1 Success response example

After the credential callback is sent successfully, an example of the subscription information returned by RevenueCat:

```json
{
  "request_date": "2026-09-17T07:47:20Z",
  "subscriber": {
    "entitlements": {
      "chatgpt_go": {
        "product_identifier": "oai_chatgpt_go_1000_1m",
        "purchase_date": "2026-09-17T07:38:26Z",
        "expires_date": "2026-10-17T07:38:26Z"
      }
    },
    "original_app_user_id": "46ee5a3b-97d3-4da0-baa8-061ecf9f1b25",
    "subscriptions": {
      "oai_chatgpt_go_1000_1m": {
        "price": { "amount": 8.0, "currency": "USD" },
        "store": "app_store",
        "period_type": "normal",
        "ownership_type": "PURCHASED",
        "original_purchase_date": "2026-09-17T07:38:26Z",
        "expires_date": "2026-10-17T07:38:26Z"
      }
    }
  }
}
```

### 5.2 Key response fields

| Field | Description |
|------|------|
| `entitlements` | Subscription benefits currently owned by the account |
| `original_app_user_id` | The account ID the subscription is bound to (should be the target account after transfer) |
| `expires_date` | Subscription expiration time |
| `ownership_type` | `PURCHASED` means purchased |

---

> **Last updated**: 2026-09-17
