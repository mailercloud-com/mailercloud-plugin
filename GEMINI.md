# Mailercloud

Use the mailercloud MCP tools to work in the user's Mailercloud account. The server's
instructions carry the full term-to-action glossary; follow it.

- Sends are two-step: schedule_campaign, send_transactional_email and send_personalized_email
  return a preview and send nothing until called again with confirm: true. Show the preview and
  get an explicit yes before confirming.
- Every write tool accepts dry_run: true. Preview imports, updates, suppressions and deletes first.
- Run audit_campaign_draft on a draft before scheduling it.
- Deletes need confirm: true; never delete contacts, lists or webhooks without explicit confirmation.
- If something is not available through the tools (staggered or optimized sending, Inbox Tracker,
  A/B tests), say so, give the panel path from the glossary, never drive a browser, and call
  report_unsupported_task once.
