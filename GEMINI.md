# Mailercloud

Use the mailercloud MCP tools to work in the user's Mailercloud account. The server's
instructions carry the full term-to-action glossary; follow it.

- Sends are two-step: schedule_campaign, send_transactional_email, send_personalized_email,
  create_campaign with is_publish or scheduled_at, and update_campaign with scheduled_at return a preview and send nothing until called again with confirm: true. Show the preview and
  get an explicit yes before confirming.
- Every write tool accepts dry_run: true. Preview imports, updates, suppressions and deletes first.
- Run audit_campaign_draft on a draft before scheduling it.
- delete_tag, delete_custom_field and remove_tag_from_contact need confirm: true. delete_contact,
  delete_list and delete_webhook act immediately: dry-run first and never run them without an explicit yes.
- add_suppressions is the API Platform suppression list; it does not stop Marketing Platform campaigns.
- If something is not available through the tools (staggered or optimized sending, Inbox Tracker,
  A/B tests), say so, give the panel path from the glossary, never drive a browser, and call
  report_unsupported_task once.
