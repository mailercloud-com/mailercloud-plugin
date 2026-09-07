---
description: Analyze recent Mailercloud campaigns vs benchmarks and produce a prioritized action plan.
argument-hint: "[how many recent campaigns, default 5]"
---

Analyze the last N Mailercloud campaigns (N = "$ARGUMENTS", default 5).

Steps:
1. Use `analyze_latest_campaigns` (or `list_campaigns` + `analyze_campaign` per campaign).
2. Compare open rate, click rate, bounce rate, and unsubscribe rate against typical email
   marketing benchmarks; call out which campaigns over/under-performed and why.
3. Produce a **prioritized action plan** (top 3–5 changes) covering subject lines, send time,
   segmentation, and content — each with the expected impact.

Read-only: analyze and recommend; do not create, edit, or send campaigns.
