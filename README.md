# 💰 AppScript Personal Finance Tracker

> A fully serverless personal finance ecosystem built on **Google Apps Script**, powered by a **Telegram bot interface** and backed by **Google Sheets** as a live database. Designed to run entirely on free-tier infrastructure — no servers, no subscriptions, no monthly costs.

---

## 📌 Overview

Managing personal finances shouldn't require a paid app or a complicated setup. This project turns a Google Sheet into a full financial management system, accessible from anywhere through Telegram. Expenses and income all flow into a single spreadsheet — automatically categorised, tracked by month, and queryable on demand.

Built to work as a robust, lite alternative to popular workflow apps such as N8N that connects various APIs together. The project offers a quick, no frills way of tracking personal finances, whilst being open enough for relevant tinkering and expimentation.

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
   ├── AI Categorisation (OpenRouter API via UrlFetchApp; Default is Gemini)
   ├── Inline Keyboard Builder
   └── Sheet Writer (appendRow / setValues / getRange)
        │
        ▼
Google Sheets (Live Database)
   ├── Expenditure Table
   ├── Income Table
   ├── Trade Journal (IBKR)
   └── Settings Sheet


```

---

## 🛠️ Tech Stack

| Layer | Tool | Cost |
|---|---|---|
| Bot interface | Telegram Bot API | Free |
| Backend runtime | Google Apps Script | Free |
| Database | Google Sheets | Free |
| AI categorisation | OpenRouter (free models) | Free |

**Total monthly infrastructure cost: $0**

---

## 🚀 Setup

### Prerequisites
- Google account (Sheets + Apps Script)
- Telegram account + a bot token from [@BotFather](https://t.me/BotFather)
- OpenRouter API key ([openrouter.ai](https://openrouter.ai)) — free tier is sufficient
- Google AI Studio API key ([ai.google.dev](https://ai.google.dev)) — for receipt OCR

### 1. Google Sheet Structure

1. You can find the Google Sheet Template via this link: PLACEHOLDER , make a copy
2. Create a new page and rename it with the current year,"202x"
3. Find the page "Main Template", click on any cell and press ctrl + A and ctrl + c to select and copy every cell, paste everything into the sheet named after the current year
5. Press **Share** → Change access to **Anyone With Link: Editor**


### 2. Credential Setup
1. To set up the various credentials, click on the GUI button "Telegram Bot", there will be 5 credentials that you will need to fill within the sheet GUI
2. The 'Bot Token' can be found by creating a telegram bot via the "Botfather" bot, the generated token will be an alphanumeric string with a set of numbers followed by a colon ":" and then a long string of jumbled letters and numbers
3. To setup the webhook url, Open your Google Sheet → **Extensions → Apps Script**, press **deploy** →  **new deployment** → Select "Web App" under 'Select type' and Select 'Anyone' when prompted 'who has access' → **Deploy** (Agree to all of the Services) → copy the Deployment URL of the Web App and fill it in the GUI of the Google Sheet
4. To setup the AI API, create an account with Openrouter, create a project and save the API key, fill it in the Google Sheet GUI. 
5. The spreadsheet id can be found 'https://docs.google.com/spreadsheets/d/HERE!It is a long string of numbers and integers/edit?gid=2031327487#gid=2031327487'in the url of the Google Sheet database
5. Lastly, fill in the Google Sheets url from the browser into the form
6. Press "Set up Telegram Bot Integration ⚙️" to setup the connection via the Bot Token provided
7. Lastly, press "Check Webhook Status 🔍" to validate the setup credentials



---

## 📱 Usage

### Options Menu
Press **/options** to open the options menu, there will be options to record purchases, income records as well as functions to check daily and monthly expenses

### Logging an Expense

Select the 
```
Bot:
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
