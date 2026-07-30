# 💰 AppScript Personal Finance Tracker

> A fully serverless personal finance ecosystem built on **Google Apps Script**, powered by a **Telegram bot interface** and backed by **Google Sheets** as a live database. Designed to run entirely on free-tier infrastructure — no servers, no subscriptions, no monthly costs.

link to Google Sheets: https://docs.google.com/spreadsheets/d/1PYpEZT3NYxPltFvUyPm_AbeOx4J4V6J8NUfR-vdbfQ4/edit?gid=0#gid=0
[Note: Copying the Google Sheet will also copy the relevant appscript files over to the users Google account]

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

### 🧾 Receipt OCR Pipeline (via n8n + Gemini Vision)
- Photo of a receipt sent to Telegram → n8n receives image → Gemini Vision extracts line items and total → structured data written to Google Sheets
- Eliminates manual entry for physical receipts — photograph and forget
- Handles supermarkets, hawker receipts, and retail invoices

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
- Google AI Studio API key ([ai.google.dev](https://ai.google.dev)) — for free AI request

### 1. Google Sheet Structure

1. You can find the Google Sheet Template via this link: PLACEHOLDER , make a copy
2. Create a new page and rename it with the current year,"202x"
3. Find the page "Main Template", click on any cell and press ctrl + A and ctrl + c to select and copy every cell, paste everything into the sheet named after the current year. Check that all the formulas in the cells are copied over.
5. Press **Share** → Change access to **Anyone With Link: Editor**


### 2. Credential Setup
1. To set up the various credentials, click on the GUI button "Telegram Bot", there will be 5 credentials that you will need to fill within the sheet GUI
2. The 'Bot Token' can be found by creating a telegram bot via the "Botfather" bot, the generated token will be an alphanumeric string with a set of numbers followed by a colon ":" and then a long string of jumbled letters and numbers
3. To setup the webhook url, Open your Google Sheet → **Extensions → Apps Script**, press **deploy** →  **new deployment** → Select "Web App" under 'Select type' and Select 'Anyone' when prompted 'who has access' → **Deploy** (Agree to all of the Services) → copy the Deployment URL of the Web App and fill it in the GUI of the Google Sheet
4. To setup the AI API, create an account with Openrouter, create a project and save the API key, fill it in the Google Sheet GUI. 
5. The spreadsheet id is the set of characters between'https://docs.google.com/spreadsheets/d/' and `/edit?gid=XXXXXXXXX#gid=XXXXXXXXX'in the url of the Google Sheet database
5. Lastly, fill in the Google Sheets url from the browser into the form
6. Press "Set up Telegram Bot Integration ⚙️" to setup the connection via the Bot Token provided
7. Press "Check Webhook Status 🔍" to validate the setup credentials
8. Press **Deploy** on the App Script Menu → **Manage Deployment** → Change the Version to the latest Version
9. Lastly, type anything into the chat for the ```dopost()``` function to save the telegram Chat ID for automated Alerts

### 3. Maintenance Notes
1. To create a database for a new year, create a page named with the new year, the code will automatically target the specific sheet for indexing
2. Ensure that both the Google Appscript and Sheets sharing access are both able to be edited by "Anyone"
3. If more income streams or expenditure categories are to be added. Simply add more headers next to the ones provided and drag the existing cell formulas over to include the newly added columns. Once the formulas are applied, the code will automatically index the new categories for analysis and tracking

---

## 📱 Usage

### Options Menu
Press **/options** to open the options menu, there will be options to record purchases, income records as well as functions to check daily and monthly expenses

<img width="313" height="350" alt="image" src="https://github.com/user-attachments/assets/b50a29ac-651a-4f4e-b504-f26f37db8278" />

### Logging an Expense

<img width="293" height="171" alt="image" src="https://github.com/user-attachments/assets/5c81c299-19b4-41a3-9afb-60d4c5569d39" />


### Logging Income

<img width="292" height="213" alt="image" src="https://github.com/user-attachments/assets/efa326d4-0363-4c19-8369-b021d95dafdd" />


### Checking Monthly Expenditure and Income Balance Sheet (💲Expenditure Breakdown)

<img width="364" height="415" alt="image" src="https://github.com/user-attachments/assets/b7b98289-9a76-44cf-9ce4-dfc1335c50e6" />


### Checking Daily Expenditure

<img width="338" height="231" alt="image" src="https://github.com/user-attachments/assets/cea616d9-9bea-4297-a0b7-443440d40cb3" />


### Checking Raw Expenditure Data

<img width="266" height="573" alt="image" src="https://github.com/user-attachments/assets/6eb1ce83-092e-4064-9ad0-276f97722da6" />

---


## 📁 Project Structure

```
/
├── main.gs                  ← Main webhook handler and state machine
├── config.gs                ← All relevant config functions
├── setup.gs                 ← Functions for setting up credentials and webhook
├── mcp_main.gs              ← OpenRouter categorisation calls
```

---

## 🤝 Contributing

This project is primarily built for personal use, but issues and PRs are welcome — especially around:
- Additional broker integrations (Tiger Brokers, Moomoo,IBKR flex query)
- Google Sheets formula improvements for reporting

---

## 📄 License

MIT License — free to use, fork, and adapt.

---

<div align="center">
  Built with Google Apps Script · Powered by Telegram · Zero monthly cost
</div>
