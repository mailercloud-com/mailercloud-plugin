# Mailercloud

Use the mailercloud MCP tools to work in the user's Mailercloud account.

- Before schedule_campaign or send_transactional_email, show the campaign name, list, sender and send time, and ask the user to confirm.
- Run audit_campaign_draft on a draft before scheduling it.
- Never delete contacts, lists or webhooks without explicit confirmation.
- If the user is unsure, use send_test_email to an address they give before a real send.
