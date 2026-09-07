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
- **Write (confirm first):** `create_*`, `update_*`, `upsert_contact`, `batch_create_contacts`,
  `schedule_campaign`, `send_test_email`, `send_transactional_email`, `toggle_webhook`.
- **Destructive (double-confirm):** `delete_contact`, `delete_list`, `delete_webhook`.

## Core rules
1. **Never send or schedule without explicit confirmation.** `schedule_campaign`,
   `send_test_email`, and `send_transactional_email` reach real recipients. Show what will be
   sent, to which list/how many recipients, and from which sender — then wait for a yes.
2. **Always audit a draft before sending** (`audit_campaign_draft`) — see the audit checklist below.
3. **Deletes are irreversible.** For `delete_*`, restate exactly what will be removed and require a
   second confirmation. Prefer archiving/deactivating over deleting when possible.
4. **Respect the authenticated account's scope.** The connector acts as the signed-in Mailercloud
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
