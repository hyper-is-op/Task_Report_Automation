# 📋 Task Report Automation — n8n Workflow

An n8n workflow that lets you manage daily task reports via **Telegram**, backed by **Google Sheets**, with optional sharing to **Google Chat**.

---

## 🧭 Overview

This workflow is triggered by a Telegram message. It presents an interactive form to the user and routes to one of five actions based on their selection:

| Action | Description |
|---|---|
| `get last 10 entries` | Fetches entries from the last 10 days in the most recent sheet |
| `get entry by date` | Looks up a specific date's task entry |
| `post` | Posts the latest Google Sheets URL to a Google Chat space |
| `insert or update` | Adds or updates a task entry for a given date |
| `exit` | Exits the workflow and sends a goodbye message |

The workflow auto-times out after **5 minutes** of inactivity (Asia/Kolkata timezone).

---

## 🗺️ Workflow Diagram

```
Telegram Trigger
      ↓
  URL Node (set spreadsheet & chat config)
      ↓
  Get all sheet names (HTTP → Sheets API)
      ↓
  Options to User (Telegram send-and-wait form)
      ↓
  Switch Case
  ├── get last 10 entries → Get latest sheet → Filter last 10 days → Telegram message
  ├── get entry by date  → Format input → Fetch sheet → Match date → Telegram (found/not found)
  ├── post               → Post URL to Google Chat → Telegram confirmation
  ├── insert or update   → Format input → Check sheet exists (create if not) → Append/update row → Telegram success
  └── exit               → Telegram goodbye message
```

---

## ⚙️ Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- A **Telegram Bot** (via [@BotFather](https://t.me/BotFather))
- A **Google account** with:
  - Google Sheets OAuth2 credentials
  - Google Chat OAuth2 credentials (if using the `post` action)
- A target **Google Spreadsheet** where task reports will be stored

---

## 🔐 Credentials Required

| Credential | Used By |
|---|---|
| `telegramApi` | Telegram Trigger, all Telegram send nodes |
| `googleSheetsOAuth2Api` | All Google Sheets nodes + HTTP request to Sheets API |
| `googleChatOAuth2Api` | Google Chat message node (`post` action) |

---

## 🔧 Setup Instructions

### 1. Import the workflow

In your n8n instance:

1. Go to **Workflows → Import from file**
2. Upload `Task_Report_Automation.json`

### 2. Configure the URL node

Open the **URL node** and update these values to match your setup:

```
original_URL_shared_link    → Your Google Sheets shared URL
original_URL_copied_link    → Your Google Sheets direct tab URL
Google Chat Space ID        → Your Google Chat space ID (e.g. spaces/XXXXXXXX)
original_URL_sheet_name     → Sheets API URL for fetching sheet names
original_URL_full_data      → Sheets API URL for fetching all grid data
```

> The Spreadsheet ID is embedded in all these URLs — replace `1Fnatbxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx6fzyC85rU` with your own spreadsheet's ID.

### 3. Set up credentials

1. Go to **Settings → Credentials** in n8n
2. Create/connect your Telegram, Google Sheets, and Google Chat credentials
3. In the imported workflow, assign your credentials to each relevant node

### 4. Activate the workflow

Toggle the workflow to **Active** — the Telegram webhook will start listening for messages.

---

## 📊 Google Sheets Structure

Each sheet is named by **month and year** (e.g. `May 2026`, `April 2026`).

Each sheet has two columns:

| Column | Format | Example |
|---|---|---|
| `Date(M/D/YYYY)` | M/D/YYYY | `5/5/2026` |
| `Task` | Free text | `Completed API integration` |

The workflow automatically creates a new sheet for the current month if one doesn't exist yet.

---

## 🤖 How to Use

1. Send **any message** to your Telegram bot
2. The bot replies with an interactive form — select an action from the dropdown
3. Fill in the **Date** and/or **Task** fields as needed
4. Submit — the bot confirms the result via Telegram

**Tip:** If you don't interact within 5 minutes, the workflow exits automatically.

---

## 🔀 Workflow Logic Details

### `get last 10 entries`
- Identifies the **most recently named** sheet (by parsing sheet tab names as dates)
- Reads all rows, filters to those within the last 10 calendar days
- Returns each entry as a separate Telegram message

### `get entry by date`
- Parses the form date input (`YYYY-MM-DD` → `M/D/YYYY` for matching, Google Sheet Default linked with Google Account Default)
- Reads the sheet for the corresponding month
- Normalizes date formats before comparing to handle leading-zero differences
- Sends either an "entry found" or "not found" Telegram message

### `post`
- Posts the Google Sheets shared URL directly into the configured Google Chat space
- Sends a Telegram confirmation: _"Task Report Posted in 'Team Discussion'"_

### `insert or update`
- Formats the date as `M/D/YYYY` and resolves the sheet name (e.g. `May 2026`)
- Checks if that month's sheet exists — creates it if not
- Uses **append or update** mode: updates an existing row if the date matches, otherwise appends
- Sends a Telegram success message with the saved date and task

### `exit`
- Immediately sends a goodbye message and stops the workflow

---

## 📁 File Structure

```
Task_Report_Automation.json   ← n8n workflow export (import this)
README.md                     ← This file
```

---

## 🛠️ Customisation Tips

- **Change the timeout:** In the `Options to USER` node, edit `resumeAmount` (default: 5 minutes)
- **Change the timezone:** In workflow Settings → Timezone (default: `Asia/Kolkata`)
- **Change the Google Chat space:** Update `Google Chat Space ID` in the URL node
- **Add more actions:** Extend the Switch Case node with new conditions and connect new branches

---

## 🧩 Nodes Used

| Node | Purpose |
|---|---|
| `n8n-nodes-base.telegramTrigger` | Listens for incoming Telegram messages |
| `n8n-nodes-base.telegram` | Sends messages and interactive forms via Telegram |
| `n8n-nodes-base.set` | Sets URL config values and formats output fields |
| `n8n-nodes-base.httpRequest` | Fetches raw sheet metadata via Sheets REST API |
| `n8n-nodes-base.switch` | Routes flow based on selected action |
| `n8n-nodes-base.googleSheets` | Reads, creates, appends, and updates sheet rows |
| `n8n-nodes-base.googleChat` | Posts messages to a Google Chat space |
| `n8n-nodes-base.code` | JavaScript logic for date parsing, filtering, and formatting |

---

## 📄 License

MIT — free to use, modify, and share.
