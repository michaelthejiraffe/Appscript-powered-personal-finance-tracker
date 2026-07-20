# 💰 AppScript Personal Finance Tracker

> A fully serverless personal finance ecosystem built on **Google Apps Script**, powered by a **Telegram bot interface** and backed by **Google Sheets** as a live database. Designed to run entirely on free-tier infrastructure — no servers, no subscriptions, no monthly costs.

---

## 📌 Overview

Managing personal finances shouldn't require a paid app or a complicated setup. This project turns a Google Sheet into a full financial management system, accessible from anywhere through Telegram. Expenses, income, trades, and portfolio data all flow into a single spreadsheet — automatically categorised, tracked by month, and queryable on demand.

Built to work around an NS schedule: async, mobile-first, and requiring minimal manual input.

---

## ✨ Features

### 🤖 Telegram Bot Interface
- Send messages directly to a bot to log financial data — no app switching, no browser required
- **Inline keyboard menus** guide the user through purchase and income flows without needing to remember command syntax
- **State machine architecture** manages multi-step flows (`awaiting_purchase`, `awaiting_income`) so the bot knows what context each reply belongs to
- Custom `ITEMNAME_PRICE` message format for rapid single-line expense entry

### 🧠 AI-Powered Categorisation
- Expense and income entries are automatically categorised using an **OpenRouter AI call** (`parseHeadersWithOpenRouter`)
- Categories are dynamically pulled from the live spreadsheet via `updateExpenditureCategories()` and `updateIncomeStreamCategories()` — adding a new category to the sheet instantly makes it available to the AI
- No hardcoded category lists — the model infers the best match from your actual tracked categories

### 💸 Expense Tracking
- Log purchases with a single message — name and price parsed automatically
- Each entry records: timestamp, item name, raw value, AI-assigned category, and month name
- Entries appended to the sheet in real time via `sheet.appendRow()`
- Confirmation message sent back to Telegram with full entry summary

### 💵 Income Tracking
- Separate income logging flow with its own category set (freelance, tutoring, dividends, trading income, etc.)
- Entries written to a dedicated income section of the sheet at a configurable fixed row offset
- Row position tracked via a settings cell (`B6`) so the income table grows automatically without overwriting other data
- Confirmation message includes source name, amount, and assigned income stream category

### 📊 IBKR Trade Journal (via n8n)
- Automated daily trade report pulled from **Interactive Brokers Flex Query API**
- Workflow: schedule trigger → request Flex report → extract reference code → wait → download XML → parse → split trades → write to Google Sheets
- Fields recorded per trade: `tradeDate`, `dateTime`, `symbol`, `description`, `quantity`, `buySell`, `tradePrice`, `tradeMoney`, `currency`, `fxRateToBase`, `tradeID`
- Deduplication via `appendOrUpdate` matched on `tradeID` — safe to re-run daily without creating duplicate rows
- Supports multi-currency trades with FX rate capture for SGD base currency reconciliation

### 🧾 Receipt OCR Pipeline (via n8n + Gemini Vision)
- Photo of a receipt sent to Telegram → n8n receives image → Gemini Vision extracts line items and total → structured data written to Google Sheets
- Eliminates manual entry for physical receipts — photograph and forget
- Handles supermarkets, hawker receipts, and retail invoices

### 📰 Financial News Digest
- Daily portfolio news digest delivered to Telegram each morning
- Covers tracked holdings: DBS, UOB, Keppel, CapitaLand Ascendas, Frasers Logistics, NetLink, Sasseur, AIMS APAC, VWRA, CSPX, Kraken Robotics
- AI filters noise and flags only material events: earnings surprises, dividend changes, analyst downgrades, MAS policy actions
- SGX dividend announcement tracker writes ex-dividend dates and amounts to Sheets and sends pre-date alerts

### 📅 Monthly Tracking & Reporting
- Every entry tagged with current month name via `getCurrentMonthName()`
- Enables month-over-month breakdown of spending, income, and trade P&L within the same sheet
- Settings sheet (`getSettingsSheet()`) stores global configuration: row offsets, category lists, thresholds

---

## 🏗️ Architecture

```
You (Telegram message)
        │
        ▼
Telegram Bot API
        │
        ▼
Google Apps Script (Webhook handler)
   ├── State Machine (awaiting_purchase / awaiting_income)
   ├── AI Categorisation (OpenRouter API via UrlFetchApp)
   ├── Inline Keyboard Builder
   └── Sheet Writer (appendRow / setValues / getRange)
        │
        ▼
Google Sheets (Live Database)
   ├── Expenditure Table
   ├── Income Table
   ├── Trade Journal (IBKR)
   └── Settings Sheet

n8n (Self-hosted on Oracle Cloud Free Tier)
   ├── IBKR Flex Query → Trade Journal
   ├── Receipt OCR Pipeline (Gemini Vision)
   ├── Daily Portfolio News Digest
   └── SGX Dividend Tracker
        │
        ▼
Cloudflare Tunnel (HTTPS webhooks, zero open ports)
```

---

## 🛠️ Tech Stack

| Layer | Tool | Cost |
|---|---|---|
| Bot interface | Telegram Bot API | Free |
| Backend runtime | Google Apps Script | Free |
| Database | Google Sheets | Free |
| AI categorisation | OpenRouter (free models) | Free |
| Receipt OCR | Gemini Vision API (Google AI Studio) | Free |
| Workflow automation | n8n (self-hosted) | Free |
| Cloud hosting (n8n) | Oracle Cloud Always Free Tier | Free |
| Tunnel / HTTPS | Cloudflare Tunnel | Free |
| Trade data | IBKR Flex Query API | Free (with IBKR account) |

**Total monthly infrastructure cost: $0**

---

## 🚀 Setup

### Prerequisites
- Google account (Sheets + Apps Script)
- Telegram account + a bot token from [@BotFather](https://t.me/BotFather)
- OpenRouter API key ([openrouter.ai](https://openrouter.ai)) — free tier is sufficient
- Google AI Studio API key ([ai.google.dev](https://ai.google.dev)) — for receipt OCR
- IBKR account with Flex Query access — for trade journal automation

### 1. Google Sheet Structure

Create a Google Sheet with the following layout:

| Section | Location | Purpose |
|---|---|---|
| Expenditure table | Starting row 1 | Purchase entries (date, name, amount, category, month) |
| Income table | Starting row 55 | Income entries (same fields, column G onwards) |
| Settings sheet | Separate tab | Configuration: category lists, row counters, thresholds |

In the Settings sheet, set **cell B6** to `0` — this tracks the income row offset and auto-increments with each entry.

### 2. Apps Script Setup

1. Open your Google Sheet → **Extensions → Apps Script**
2. Paste the project files into the editor
3. Add Script Properties (⚙️ Project Settings → Script Properties):

```
TELEGRAM_BOT_TOKEN   = your_bot_token
OPENROUTER_API_KEY   = your_openrouter_key
TELEGRAM_CHAT_ID     = your_chat_id
GEMINI_API_KEY       = your_gemini_key
```

4. Deploy as a **Web App** (Execute as: Me, Access: Anyone)
5. Copy the deployment URL

### 3. Register the Telegram Webhook

```bash
curl "https://api.telegram.org/bot<YOUR_TOKEN>/setWebhook?url=<YOUR_APPS_SCRIPT_URL>"
```

### 4. n8n Workflows (Optional)

For trade journal, receipt OCR, and news digest automation:

1. Self-host n8n on Oracle Cloud free tier (see [n8n Docker setup docs](https://docs.n8n.io/hosting/installation/docker/))
2. Set up a Cloudflare Tunnel to expose your n8n instance via HTTPS
3. Import the workflow JSON files from the `/n8n-workflows` folder
4. Configure credentials: Google Sheets OAuth2, Telegram, IBKR Flex token

---

## 📱 Usage

### Logging an Expense
```
You:  /spend
Bot:  [Inline keyboard: Quick Add | Manual Entry]

You:  Lunch $8.50
Bot:  ✅ Added: Lunch ($8.50)
      Category: Food & Dining
```

### Logging Income
```
You:  /income
Bot:  [Inline keyboard: Tutoring | Dividends | Freelance | Other]

You:  Physics tutoring 80
Bot:  ✅ Added: Physics tutoring ($80.00)
      Income Source: Tutoring
```

### Receipt Scan
```
You:  [Photo of receipt]
Bot:  ✅ Receipt processed
      3 items detected — total $24.70 added to Expenses
```

### Portfolio Digest (automated, daily 7AM)
```
Bot:  📊 Morning Brief — 20 Jul 2026
      DBS ▲ +1.2%  |  Neutral — no material events
      CSPX ▲ +0.4%  |  Neutral
      Kraken Robotics 🚨  |  Bullish — new navy contract announced
```

---

## 🗺️ Roadmap

- [ ] **Monthly spending summary** — automated Telegram report on the 1st of each month
- [ ] **Budget threshold alerts** — notify when category spending exceeds a set limit
- [ ] **Options trade tracker** — dedicated sheet section for wheel strategy (CSPs + covered calls)
- [ ] **SGX dividend calendar** — auto-populate ex-dividend and payment dates from SGX announcements
- [ ] **REIT health dashboard** — monthly DPU, gearing ratio, and occupancy tracking for S-REIT holdings
- [ ] **Google Calendar integration** — sync payment due dates and tuition sessions into Calendar
- [ ] **Multi-currency P&L reconciliation** — automatic SGD conversion for USD-denominated IBKR trades
- [ ] **Notion sync** — mirror key financial summaries into a Notion dashboard

---

## 📁 Project Structure

```
/
├── Code.gs                  ← Main webhook handler and state machine
├── SheetUtils.gs            ← Sheet read/write helpers
├── TelegramUtils.gs         ← Message sending, keyboard builder
├── AIUtils.gs               ← OpenRouter categorisation calls
├── StateManager.gs          ← User state persistence (PropertiesService)
├── Config.gs                ← Constants, sheet ranges, category defaults
│
└── n8n-workflows/
    ├── ibkr-trade-journal.json      ← IBKR Flex Query → Google Sheets
    ├── receipt-ocr-pipeline.json    ← Telegram photo → Gemini → Sheets
    ├── portfolio-news-digest.json   ← Daily news → Telegram
    └── sgx-dividend-tracker.json    ← SGX RSS → Sheets + Telegram alert
```

---

## 🤝 Contributing

This project is primarily built for personal use, but issues and PRs are welcome — especially around:
- Additional broker integrations (Tiger Brokers, Moomoo)
- New n8n workflow templates
- Google Sheets formula improvements for reporting

---

## 📄 License

MIT License — free to use, fork, and adapt.

---

<div align="center">
  Built with Google Apps Script · Powered by Telegram · Zero monthly cost
</div>
