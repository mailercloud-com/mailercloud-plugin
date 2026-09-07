---
description: Audit a Mailercloud draft campaign before sending — subject, links, and deliverability risks.
argument-hint: "[campaign name or id]"
---

Audit the Mailercloud campaign referenced by "$ARGUMENTS" (if empty, list recent draft
campaigns with `list_campaigns` and ask which one).

Steps:
1. Fetch the campaign with `get_campaign` (and `audit_campaign_draft` if available).
2. Review and flag issues in these areas:
   - **Subject line & preview text** — length, clarity, spam-trigger words, personalization.
   - **Links** — broken links, missing tracking, shortened/blacklisted domains.
   - **Sender & authentication** — verified sender, SPF/DKIM/DMARC alignment (note if unknown).
   - **Content** — image-to-text ratio, missing unsubscribe/physical address, mobile rendering.
   - **List & timing** — target list size/health, send time.
3. Return a short, prioritized checklist: 🔴 must-fix, 🟡 should-fix, 🟢 optional — each with the
   concrete change to make.

Do NOT send or schedule anything. This is a read-only audit; only report findings.
