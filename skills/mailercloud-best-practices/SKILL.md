---
name: mailercloud-best-practices
description: Email marketing best practices for working with Mailercloud — how to plan, audit, send, and analyze campaigns, manage contacts and lists, and protect deliverability. Use whenever the user is building, reviewing, sending, or analyzing email campaigns, managing contacts/lists/segments, or asking about deliverability, opens, clicks, bounces, or sender reputation via Mailercloud.
---

# Mailercloud Best Practices

Guidance for using the Mailercloud MCP tools effectively. Prefer read/analysis tools first;
confirm with the user before any create/update/delete/send action.

## Tool map (by task)
- **Discover:** `list_campaigns`, `list_contact_lists`, `list_contacts`, `list_segments`,
  `list_senders`, `list_templates`/`list_template_categories`, `list_tags`, `list_webhooks`.
- **Analyze (read-only):** `analyze_campaign`, `analyze_latest_campaigns`, `compare_campaigns`,
  `campaign_health_dashboard`, `engagement_funnel`, `get_campaign_domain_report`,
  `get_inbox_tracking`, `get_account_overview`, `get_best_practices`.
- **Audit before sending:** `audit_campaign_draft`.
- **Write (dry_run first, confirm before acting):** `create_*`, `update_*`, `upsert_contact`,
  `batch_create_contacts`, `add_lists_to_campaign`, `add_suppressions`, `toggle_webhook`.
- **Send (two-step `confirm`):** `schedule_campaign`, `send_transactional_email`,
  `send_personalized_email`; `send_test_email` to user-given addresses only.
- **Guided workflows (MCP prompts):** `plan_campaign`, `audit_draft`, `monthly_review`,
  `clean_my_list`.
- **Can't do it?** Panel hand-off from the glossary, then `report_unsupported_task`.
- **Destructive (double-confirm):** `delete_contact`, `delete_list`, `delete_webhook`.

## Term map (read first)
`references/glossary.md` maps every Mailercloud term a user might say (STO, staggered send,
Inbox Tracker, exclusions, segments, double opt-in...) to one outcome: the **tool** to call, a
**panel** hand-off with the exact path, an **add-on** with its price, or **not offered**. Look a
term up there before improvising. Never try to drive the browser to finish something the tools
cannot do: say it is not available through the connector, finish everything else, give the panel
path, then call `report_unsupported_task` once.

## Core rules
1. **Sends are two-step, enforced by the server.** `schedule_campaign`, `send_transactional_email`
   and `send_personalized_email` return a preview (recipients, sender, subject, send time) and send
   nothing unless called again with `confirm: true`. Show the preview, wait for an explicit yes,
   then confirm. `send_test_email` reaches real inboxes too: confirm the addresses first.
2. **Always audit a draft before sending** (`audit_campaign_draft`) — see the audit checklist below.
3. **Preview before any write.** Every write tool accepts `dry_run: true`, which validates and shows
   what would change without changing it. Use it for imports, updates, suppressions and deletes.
4. **Deletes are irreversible.** `delete_*` and tag removal need `confirm: true`; restate exactly
   what will be removed and get a second yes. Prefer deactivating over deleting when possible.
5. **Respect the authenticated account's scope.** The connector acts as the signed-in Mailercloud
   account; don't assume access to data outside it.

## Pre-send audit checklist
- **Sender/auth:** verified sender; SPF/DKIM/DMARC aligned (flag if unverifiable).
- **Subject & preview:** concise, non-spammy, personalized where useful; preview text set.
- **Links:** all resolve; tracking intact; no shortened/blacklisted domains.
- **Content:** balanced image-to-text; working unsubscribe link; physical mailing address present;
  renders on mobile.
- **List:** correct, healthy target list/segment; suppress unsubscribed/bounced.
- **Timing:** appropriate send time for the audience's timezone.

## Deliverability guidance
- **Warm up** new domains/IPs gradually; don't blast a cold list.
- **Segment** by engagement; stop mailing long-term non-openers to protect reputation.
- **Watch bounces:** hard bounces (invalid) should be suppressed; soft/transient bounces retried,
  not suppressed. A rising bounce or complaint rate is a reputation warning — pause and investigate.
- **Authentication is mandatory** for good inbox placement — never send from an unverified domain.

## Interpreting metrics
- Compare open/click rates to the account's own historical baseline first, then to industry norms.
- A high open + low click usually means weak content/CTA; a low open usually means subject/sender/
  timing or a deliverability problem.
- Use `compare_campaigns` to isolate what changed between a strong and weak send.
