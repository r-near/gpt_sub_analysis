# iOS ChatGPT Pro 20x Activation Tutorial

<div align="center">

### 📬 Contact

✈️ **Telegram**: <a href="https://t.me/lengmeng28" target="_blank">@lengmeng28</a> &nbsp;&nbsp;|&nbsp;&nbsp; 👥 **Telegram group**: <a href="https://t.me/Geminivip1" target="_blank">@Geminivip1</a>

---

### 🌟 Recommended projects

If you are interested in this project, feel free to check out the author's other GitHub open-source projects:

🌐 **Official website**: <a href="https://codex-x.site/" target="_blank">https://codex-x.site/</a>

💻 **GitHub repository**: <a href="https://github.com/yynxxxxx/Codex-X" target="_blank">https://github.com/yynxxxxx/Codex-X</a>

⭐ **If you find this helpful, please star the project!** ⭐

</div>

---

> **⚠️ Disclaimer**: This tutorial is for learning and research purposes only. Please comply with applicable laws, regulations, and terms of service.

---

## Table of contents

1. [Prerequisites](#1-prerequisites)
2. [Network environment setup](#2-network-environment-setup)
3. [iOS jailbroken device setup](#3-ios-jailbroken-device-setup)
4. [Subscription flow in detail](#4-subscription-flow-in-detail)
5. [Key request field reference](#5-key-request-field-reference)

---

## 1. Prerequisites

Before you start, make sure all of the following are ready:

| No. | Requirement | Notes |
|:---:|------|------|
| 1 | A **jailbroken** iOS device | Sileo package manager must be installed |
| 2 | The **ChatGPT** app can be opened normally on the phone | Make sure the app version can launch normally |
| 3 | A computer with **Reqable** installed | Windows / macOS supported |
| 4 | **Clash** proxy tool installed on the computer | Used for proxying internet access |
| 5 | Computer and phone connected to the **same Wi-Fi** network | Used for man-in-the-middle packet capture |

---

## 2. Network environment setup

### 2.1 Computer setup

1. **Start Clash**
   - ✅ Make sure Clash is running normally (default port `7890`)
   - ❌ **Disable** the system proxy
   - ❌ **Disable** the virtual NIC (TUN mode)

2. **Configure Reqable as a second-level proxy**
   - Open Reqable
   - Create a **second-level proxy rule** with the following settings:
     ```
     Protocol: HTTP
     Address: 127.0.0.1
     Port: 7890 (i.e. Clash's running port)
     ```
   - Reqable itself listens on port `9000`

### 2.2 Phone setup

1. Open **Settings → Wi-Fi → the currently connected network → Proxy**
2. Select **Manual**
3. Fill in the proxy information:

| Field | Value |
|------|-----|
| Server | The computer's LAN IP address (e.g. `192.168.x.x`) |
| Port | `9000` (Reqable's listen port) |

> **💡 Tip**: You can check the local IP address in a terminal with `ifconfig` (macOS) or `ipconfig` (Windows).

### 2.3 Network path diagram

```
iPhone ──(Wi-Fi proxy 9000)──▶ Reqable ──(second-level proxy 7890)──▶ Clash ──▶ Internet
```

---

## 3. iOS jailbroken device setup

### 3.1 Install required tweaks

Install the following two tweaks from the **Sileo** store on the jailbroken device:

| Tweak | Purpose |
|--------|------|
| **SSL Kill Switch 3** | Bypass SSL Pinning so HTTPS requests can be captured |
| **Choicy** | Control which processes a tweak is injected into |

### 3.2 Configure SSL Kill Switch 3

1. Open SSL Kill Switch 3
2. Confirm the switch is **on**
3. The default global effect is fine

### 3.3 Configure Choicy

Open **Choicy → Daemons**, and enable **SSL Kill Switch 3** for the following 5 system processes:

```
✅ cloudd
✅ accountsd
✅ identityservicesd
✅ akd
✅ nsurlsessiond
```

> **⚠️ Important**: These processes handle Apple ID verification and network communication. SSL Kill Switch must be injected into them so Reqable can correctly capture App Store subscription requests.


---

## 4. Subscription flow in detail

### 4.1 Confirm prerequisites

ChatGPT Pro 20x subscription applies to the following **two types of accounts**:

- **Case A**: The Apple account **has previously subscribed** to ChatGPT services (including any of Go, Plus, or Pro 5x)
- **Case B**: An Apple ID account that has **never subscribed** to a ChatGPT membership

> **📌 Note**: For Case B (never subscribed), you need to subscribe to a **Go or Plus** membership first, then continue with the later steps. If there is a previous subscription record, you can skip this step.

### 4.2 Steps

#### Step 1: Open the subscription page

```
ChatGPT App → Settings → Subscriptions → View all plans
```

#### Step 2: Choose any plan and tap Subscribe

- Select any plan in the plan list (e.g. Plus annual)
- Tap the **Subscribe** button
- ⚠️ **Do not confirm payment immediately!**

#### Step 3: Capture packets and locate the key request

In Reqable on the computer, find the following request:

```
POST https://p44-buy.itunes.apple.com/WebObjects/MZBuy.woa/wa/buyProduct
```

#### Step 4: Rewrite the request body

Intercept and rewrite the **request body** of the request above, and change the following 3 key fields:

| Field | Original value (example: Plus annual) | Replace with (Pro 20x monthly) |
|------|------------------------|----------------------|
| `offerName` | `oai_chatgpt_plus_20000_1y` | `oai_chatgpt_pro_20000_1m` |
| `price` | `200000` | `200000` (keep unchanged) |
| `salableAdamId` | `6745416289` | `6657954405` |

> **📌 Note**: Because the ChatGPT Pro 20x plan has been disabled on the frontend page, it cannot be selected directly in the app. You need to intercept the App Store purchase request and replace the product identifier fields in the request body to complete the subscription.

#### Step 5: Confirm the subscription

After replacing the request body, allow the request through. If the operation succeeds:

```
🎉 Congratulations! A ChatGPT Pro 20x subscription confirmation dialog will appear on the phone!
```

---

## 5. Key request field reference

### 5.1 ChatGPT plan comparison table

Below are the key fields for all ChatGPT iOS subscription plans. Replace them with the corresponding plan as needed:

| Plan | `offerName` | `salableAdamId` | `mtSubscriptionAdamId` | `price` |
|------|-------------|-----------------|------------------------|---------|
| **Go** monthly $8 | `oai_chatgpt_go_1000_1m` | `6749460546` | ❌  | `8000` |
| **Plus** monthly $19.99 | `oai_chatgpt_plus_1999_1m` | `6448311597` | `6749460546` | `19990` |
| **Plus** annual $200 | `oai_chatgpt_plus_20000_1y` | `6745416289` | `6749460546` | `200000` |
| **Pro 5x** monthly $100 | `oai_chatgpt_pro_10000_1m` | `6759817441` | `6749460546` | `100000` |
| **Pro 20x** monthly $200 | `oai_chatgpt_pro_20000_1m` | `6657954405` | `6749460546` | `200000` |

> **💡 Note**: The unit of `price` is **0.001 USD** (e.g. `200000` = $200.00). The Go plan has no `mtSubscriptionAdamId` field. The other plans share the subscription group ID `6749460546`.

### 5.2 App Store purchase request

```
POST https://p44-buy.itunes.apple.com/WebObjects/MZBuy.woa/wa/buyProduct
```

**The request body is Apple Plist XML. Key fields:**

| Field | Description | Example value (Pro 20x) |
|------|------|------------------|
| `appAdamId` | ChatGPT App ID | `6448311069` |
| `bid` | Bundle ID | `com.openai.chat` |
| `offerName` | Subscription plan name | `oai_chatgpt_pro_20000_1m` |
| `price` | Price (unit: cents) | `200000` |
| `salableAdamId` | Salable product ID | `6657954405` |
| `mtSubscriptionAdamId` | Subscription group ID | `6749460546` |
| `buySubscription` | Whether this is a subscription purchase | `true` |

---

## Flow overview

```
┌──────────────────────────────────────────────────────────┐
│                    Preparation stage                     │
│  1. Jailbreak the iOS device                             │
│  2. Install SSL Kill Switch 3                            │
│  3. Install Clash + Reqable on the computer              │
│  4. Configure the network proxy path                     │
└──────────────────────┬───────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────┐
│                    Configuration stage                   │
│  1. Set Reqable second-level proxy (→ Clash:7890)        │
│  2. Point the phone Wi-Fi proxy to Reqable:9000          │
└──────────────────────┬───────────────────────────────────┘
                       ▼
┌──────────────────────────────────────────────────────────┐
│                    Execution stage                       │
│  1. ChatGPT App → Settings → Subscriptions → all plans   │
│  2. Choose any plan, tap Subscribe                       │
│  3. Intercept the buyProduct request in Reqable          │
│  4. Replace the offerName / salableAdamId fields         │
│  5. Allow the request → Pro 20x confirmation dialog 🎉   │
└──────────────────────────────────────────────────────────┘
```

---

> **Last updated**: 2026-09-17
