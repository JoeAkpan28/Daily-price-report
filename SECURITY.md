# Security

## Never commit secrets

Do not upload any of these to this repository:

- API keys (EIA, Google, Slack)
- Slack bot tokens
- Real Google Sheet links or IDs
- n8n credential IDs from your own instance
- `.env` files

The workflow file in this repo uses placeholders only (`REPLACE_WITH_...`).

## Before sharing an exported workflow

1. Open the JSON file in a text editor.
2. Search for your sheet ID, channel IDs and credential IDs.
3. Replace them with placeholders.

## If you leaked a key

1. Revoke or regenerate it right away (EIA, Slack or Google).
2. Update the credential in n8n.
3. Remove it from the repo history if needed.

## Reporting a problem

If you find a security issue in this project, please open a private message or email **[your-email@example.com]** instead of a public issue.
