# Setup Guide

A step-by-step guide to get the Daily Oil & Gas Price Briefing running. Takes about 20-30 minutes.

## Step 1: Get your EIA API key

1. Go to https://www.eia.gov/opendata/register.php
2. Fill in the form. The key is free.
3. Check your email and copy the key.

## Step 2: Create the Google Sheet

1. Open Google Sheets and create a new spreadsheet.
2. Rename the first tab to **PriceHistory**.
3. In row 1, type these headers (one per column, A to G):

   | A | B | C | D | E | F | G |
   |---|---|---|---|---|---|---|
   | date | brent_usd | wti_usd | natgas_usd | brent_change_pct | wti_change_pct | natgas_change_pct |

4. (Optional but recommended) Add one row of earlier prices so your first run can show a % change. Use a date **before** today, like `2026-10-09`. See [`sample-data/PriceHistory-sample.csv`](../sample-data/PriceHistory-sample.csv).
5. Copy the spreadsheet link from your browser.

## Step 3: Create a Slack app

1. Go to https://api.slack.com/apps and click **Create New App** → **From scratch**.
2. Open **OAuth & Permissions** and add the bot scope **`chat:write`**.
3. Click **Install to Workspace** and copy the **Bot User OAuth Token**.
4. In Slack, open the channel you want to use (for example `#oil-gas-alerts`), type `/invite @YourAppName`.

## Step 4: Import the workflow

1. Open n8n.
2. Go to **Workflows** → **Import from File**.
3. Choose `workflow/01-daily-oil-price-briefing.json`.

## Step 5: Create the credentials in n8n

Go to **Credentials** → **Create credential**.

**EIA API Key**
- Type: **Query Auth**
- Name (of the credential): `EIA API Key`
- Field *Name*: `api_key`
- Field *Value*: your EIA key

**Google Sheets account**
- Type: **Google Sheets OAuth2 API**
- Follow n8n's prompts to sign in with Google.

**Slack account**
- Type: **Slack API**
- Paste the Bot User OAuth Token from Step 3.

## Step 6: Connect the nodes

Open each node below and choose the matching credential (the red warning will disappear):

| Node | Change |
|---|---|
| EIA Crude Oil (Brent + WTI) | Pick `EIA API Key` |
| EIA Natural Gas (Henry Hub) | Pick `EIA API Key` |
| Read PriceHistory | Pick Google credential, paste your sheet link, choose the `PriceHistory` tab |
| Log to PriceHistory | Same as above |
| Slack Daily Briefing | Pick Slack credential, choose your channel |
| Slack Urgent Alert | Pick Slack credential, choose your channel |

## Step 7: Test

1. Click **Execute Workflow**.
2. Check each node's output (green = OK).
3. Check that a message arrived in Slack and a new row was added to your sheet.

### To test the alert

1. In the **Compute % Change** node, temporarily change `THRESHOLD_PCT = 3` to `THRESHOLD_PCT = 0.01`.
2. Run the workflow again. You should get the urgent alert.
3. Change it back to `3`.

## Step 8: Turn it on

Switch the **Active** toggle (top right) to on. It now runs every weekday at 07:00.

### Time zone

n8n uses its own time zone setting. For Docker, set `GENERIC_TIMEZONE` (and `TZ`) to your zone, for example `Africa/Lagos`, then restart n8n.

## Troubleshooting

| Problem | Fix |
|---|---|
| EIA node returns 403 or "invalid API key" | Check the credential: Name must be `api_key`, and the key must be correct |
| EIA node returns no data | Check the series codes: `RBRTE` (Brent), `RWTC` (WTI), `RNGWHHD` (Henry Hub gas) |
| Google Sheets "sheet not found" | Make sure the tab is named exactly `PriceHistory` |
| Slack `not_in_channel` | Invite your app to the channel with `/invite @YourAppName` |
| Slack `invalid_auth` | Re-paste the Bot User OAuth Token |
| % change shows `n/a` | There is no earlier row in the sheet yet. Add one |
| Runs at the wrong time | Fix the n8n time zone (see Step 8) |
| Import fails | Use the latest n8n version and the original file from this repo |
