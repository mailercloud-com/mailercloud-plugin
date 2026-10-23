<!-- Copied from mailercloud-com/mcp-server docs/glossary.md. Do not edit here: change the source and re-copy on each plugin release. The live server also sends this text as its instructions. -->

# Mailercloud term-to-action glossary

Source of truth for how an AI assistant (Claude, ChatGPT, Cursor, Gemini, Claude Code plugin) should interpret a Mailercloud term and what it should do about it. Rendered into: the MCP server `Instructions` string, the `mailercloud-best-practices` plugin skill, and the help-center article "Connect Mailercloud to AI assistants".

Version 0.5, 2026-10-23. Owner: Product. Built from the Mailercloud panel, the help center, the pricing pages, the public API reference at apidoc.mailercloud.com and the live connector tools; plan gating and feature availability confirmed by product.

## When the user asks what you can do

Call `get_account_overview` and answer from its "What you can do with Mailercloud here" section, not from the tool list. Cover every product, including those with few or no tools here: Marketing Platform, API Platform (transactional email), Email Verifier, and the add-ons (Inbox Tracker, Dedicated IP, Email Marketing as a Service). For each, say what works from chat and what is done in the app, with the place in the app. The `getting_started` prompt does the same and then offers first tasks.

## How to read a row

Every term resolves to exactly one **Outcome**:

| Outcome | Meaning | What the assistant does |
| --- | --- | --- |
| **TOOL** | A connector tool covers it today | Call the tool named in **Action** with the listed parameters. Confirm before any send, delete or suppression; preview writes with `dry_run: true`. |
| **WRAP** | The public API has it but the connector has no tool yet | Say it is not available in the connector yet, offer the panel path, and call `report_unsupported_task`. Developers can use the API endpoint named in **Action**. |
| **PANEL** | Panel-only; no public API | Say it is not available through the connector, give the exact panel path in **Action**, finish everything else the user asked for, then call `report_unsupported_task`. |
| **UPGRADE** | Available only on a higher plan or as an add-on | Explain the plan or add-on in **Plan**, link to Settings > Billing & Usage > Change plan, and never fail silently. |
| **INFO** | A concept, status or report value, not an action | Explain it; where a read tool exists, use it. |

Behaviour rules set by product on 2026-10-16:

- When a user hits a plan limit (contacts, monthly emails) or asks for an add-on, name the plan or add-on **and the price** (Premium from $10 per month for 5,000 contacts, 30% off annual; Inbox Tracker $120 per month; Dedicated IP $50 per IP per month) and link to Change plan or Add-ons.
- When a client asks to "clean" or "verify" a list, suggest the Email Verifier and state that it uses paid credits (1 credit per address; marketing subscribers get 100 free).
- Features marked not available (per-recipient timezone delivery, autoresponder campaigns, campaign archive, saved blocks, AI writer inside the builder, tracking domains, test automation, resend to non-openers) are stated as not offered, with the nearest alternative, never as "coming soon".

Behaviour rules added 2026-10-19 (connector guardrails):

- **Sends and deletes are two-step.** `schedule_campaign`, `send_transactional_email` and `send_personalized_email` send nothing on the first call, and `delete_tag`, `delete_custom_field`, `remove_tag_from_contact`, `delete_list`, `delete_contact`, `delete_webhook`, `add_suppressions`, `update_segment` and `toggle_webhook` (when pausing) change nothing: without `confirm: true` they return a preview (campaign or message, sender, subject, approximate recipients, send time). Show the preview, get an explicit yes, and only then call again with `confirm: true`. Never set `confirm: true` on the first call.
- **Every write tool supports `dry_run: true`** (`report_unsupported_task` excepted: it only logs). It validates the input and returns what would change without changing anything. Use it to preview creates, updates, deletes and suppressions before running them for real.
- **Report every hand-off.** After any PANEL, WRAP or not-offered hand-off, or a term this glossary does not cover, call `report_unsupported_task` once (request in the user's words without contact data, the term, the outcome, the panel path you gave). It changes nothing in the account.
- Tools never take an API key as input. Authentication is the connector sign-in; never ask the user to paste a key into the chat.

Standard hand-off sentence for WRAP and PANEL: *"[Feature] is not available through the connector yet. I have [done the rest]. To set it, open [panel path] in Mailercloud."* Never try to drive the browser to do it.

Plan column: **Free**, **Premium** (paid marketing plan), **Enterprise**, **Add-on** (Inbox Tracker, Dedicated IP, Email Marketing as a Service, Verifier credits), **API plan** (Email API Basic/Pro/Platform), **?** = gating not stated in docs, confirm with product.

---

## 1. Products and navigation

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Marketing Platform | Email marketing, the panel | Campaigns, audience, automation, templates, forms | INFO | Most connector tools act here | All plans |
| API Platform | Email API, transactional email, SMTP relay | Separate product for application email | INFO | `send_transactional_email`, suppression tools | API plan |
| Email Verifier | Verifier, list cleaning | Separate product that scores addresses | WRAP | Verifier API (`Verify Emails`, bulk file endpoints); panel: platform switcher > Email Verifier | Add-on credits |
| Platform switcher | Switch product | Top-of-sidebar toggle between the three products | PANEL | Sidebar top | All plans |
| Dashboard | Home, KPIs | Open, click, unsubscribe rates, sent graph, recent campaigns | TOOL | `campaign_health_dashboard`, `get_account_overview` | All plans |
| Account overview | Plan, contacts used, limits | Current plan and contact counts. It does not report emails sent, API quota, add-ons or verifier credits | TOOL | `get_account_overview`; emails used, add-ons and credits: Settings > Billing & Usage | All plans |
| MCP server | Connector, Claude connector, AI tools | This integration at `mcp.mailercloud.com/mcp` | INFO | Help: "Connect Mailercloud to AI assistants" | All plans |

## 2. Campaigns: create and content

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Regular campaign | Normal campaign, newsletter, broadcast, blast | One email to chosen lists, segments and tags | TOOL | `create_campaign` (name, subject, list_ids, html or plain_text, preheader, sender, reply, segments, tag_ids, exclusions, frequency_cap, pause_rules, rules). Creates a draft by default; `is_publish: true` or a `scheduled_at` publishes it, so those calls return a preview first and need `confirm: true` | All plans |
| Campaign name | Internal name | Label shown in the campaign list, not to recipients | TOOL | `create_campaign.name`, `update_campaign` | All plans |
| Subject line | Subject | Email subject | TOOL | `create_campaign.subject` | All plans |
| Preview text | Preheader, pre-header | Snippet after the subject in the inbox, max 150 chars | TOOL | `create_campaign.email_preheader` | All plans |
| Sender | From name, from address, sender ID, from id | Verified sending identity | TOOL | `list_senders` then `create_campaign.sender_id` or sender_name + sender_email; auto-resolved when only one exists | All plans |
| Reply-to | Reply ID, reply email | Address that receives replies; defaults to sender | TOOL | `list_reply_emails`, `create_campaign.reply_id` or reply_email | All plans |
| HTML content | Creative, email body, design | The email HTML | TOOL | `create_campaign.html`; or `create_template` then reference | All plans |
| Plain text version | Text version | Text alternative to HTML | TOOL | `create_campaign.plain_text` | All plans |
| Drag and drop builder | DnD builder, email builder, editor, DD-builder | Visual editor with structures, blocks, personalisation | PANEL | Templates > create, or Campaign wizard > Creative > Design | All plans |
| Custom HTML editor | Code your own, paste code | Write or paste raw HTML in the panel | TOOL | Same as HTML content via the connector; panel: Templates > Code your own | All plans |
| Rich text editor | Copy-paste editor | Simple WYSIWYG creative | PANEL | Wizard > Creative | All plans |
| Template gallery | My templates, saved templates | User's saved templates | TOOL | `list_templates` (page, limit, template_type, status, category_id, search_name, sort_field, sort_order; no content), then `get_template` (template_id or name) to read one; panel: Templates | All plans |
| Template categories | Gallery categories | Grouping of templates | TOOL | `list_template_categories` | All plans |
| Layouts and themes | Pre-built layouts, themed templates | Starter designs | PANEL | Templates > Create > Layout / Theme | All plans |
| MC templates | Mailercloud templates, ready-to-use templates | Curated system catalogue | PANEL | Templates > MC template | All plans |
| Import HTML / Import ZIP | Upload template, Stripo export, Tabular export | Bring a design from outside | PANEL (ZIP); TOOL (HTML) | `create_template.html` for plain HTML; ZIP: Templates > Create > Import | All plans |
| Create template | Save template | Store reusable HTML | TOOL | `create_template` (name, html, plain_text, category_id), `update_template` | All plans |
| Personalisation | Merge tags, personalization tags, custom field tags | Insert contact fields into content | TOOL | Use field keys from `list_custom_fields` inside html | All plans |
| Spintax | Content variation | Randomised wording in content | INFO | Supported in content; help: "Spintax" | All plans |
| Dynamic URL | Personalised links | Links built from contact fields in the builder | PANEL | Builder > link settings | All plans |
| Image upload | Content studio, hosted image, CDN image | Upload an image and get a hosted URL | PANEL | Builder > Image block > Upload | All plans |
| Prebuilt sections | Content templates, banners, testimonials, offers | Ready-made sections in the builder (there are no user-saved blocks) | PANEL | Builder > left sidebar | All plans |
| AI subject line generator | Cloudline AI, CloudLine, AI subject | Prompt-based subject suggestions, 11 languages | PANEL | Wizard > Creative > AI subject; no API | All plans |
| Brand kit | Brand profile, brand identity, apply brand | Logo, colours, fonts, footer applied to designs, DOI, unsubscribe and forms | PANEL | Settings > Account > Brand Kit; Builder > Apply brand | All plans |
| AMP email | AMP, interactive email | AMP version with HTML fallback; requires a one-time access request | PANEL | Wizard > Creative > Request AMP access; API: `send_transactional_email` version 2.0 for transactional only | All plans (access request) |
| View in browser link | Web version | Adds a browser-view link | TOOL | `update_campaign.viewin_browser` on a draft; panel: Wizard > Creative | All plans |
| Legal disclaimer / footer | Footer, physical address | Footer compliance text | PANEL | Wizard > Creative | All plans |
| Preference center link | Manage preferences | Link to the hosted preference page | PANEL | Wizard > Creative; Settings > Preference Center | All plans |
| Campaign tag | Campaign tags, campaign label | Label on a campaign, filterable in reports, segments and automation | TOOL | `create_campaign.tag_ids`; tag CRUD in Settings > Campaign tag | All plans |
| Track opens | Open tracking, open pixel | Toggle open pixel on this campaign | PANEL | Not in the public `/campaign/save` schema; panel: Wizard > Creative | All plans |
| Track clicks | Link tracking, click tracking | Rewrite links for click tracking | PANEL | Same as above | All plans |
| Conversion tracking | Conversion pixel, postback, tracking code | Count conversions from clicks via pixel or postback | PANEL | Settings > Conversion Pixel; Wizard > Creative > Conversion tracking | All plans |
| Inbox Tracker on a campaign | Inbox placement toggle, seed test | Inject seed addresses to measure inbox vs spam placement | PANEL (no API field) | Wizard > Creative or Review > Inbox Tracker toggle; results via `get_inbox_tracking` | Add-on |
| Unsupported scripts removed | Script stripping | Scripts stripped from HTML automatically | INFO | Help: "Unsupported scripts removed automatically" | All plans |

## 3. Campaigns: recipients and send rules

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Send to | Recipients, audience selection, target, "sent to options" | Lists, segments and tags the campaign goes to | TOOL | `create_campaign.list_ids`, `segments`; add or remove lists on a draft with `add_lists_to_campaign` / `remove_lists_from_campaign` (campaign_id, list_ids; merged with the current lists) | All plans |
| Add a list to a draft | List addition, add more recipients, remove a list | Extend or trim recipients of an existing draft | TOOL | `add_lists_to_campaign` / `remove_lists_from_campaign` (campaign_id, list_ids, dry_run): reads the draft's current lists and keeps them. Works on Draft and Duplicate campaigns that send to lists only; a campaign that sends to a segment is refused, because a list update clears the segment, so change those in the panel (Choose contacts) | All plans |
| Exclude | Do not send to, exclusions, suppress list from send | Remove lists, segments or tags from the recipients | TOOL (at creation) | `create_campaign.exclude_list_ids`, `exclude_segments`, `exclude_tag_ids`; on an existing draft change exclusions in the panel (Choose contacts), the update API does not accept them | All plans |
| Custom conditions for audience | Custom filter at send time | Ad-hoc conditions in Choose contacts | PANEL | Wizard > Choose contacts > Custom conditions | All plans |
| Estimated contacts | Recipient estimate, target summary, campaign reach | Deduplicated count after exclusions | PANEL | Wizard > Choose contacts > Target summary; `get_list_details` gives per-list counts only | All plans |
| Sending limit | First N / last N, send order, rules | Cap the send at N contacts taken first or last | TOOL | `create_campaign.rules` (limit, contacts_order first or last); on a draft `update_campaign.rules`; panel: Choose contacts > Additional settings | All plans |
| Frequency capping | Frequency cap, skip recently emailed, mailing frequency | Skip contacts emailed within the account window | TOOL (at creation) | `create_campaign.frequency_cap`; on an existing draft use the panel; account window in Settings > Preferences | All plans |
| Pause rules | Smart test rule, pause limit, pause percentage, wait time, send to 1 to 50% first | Send a slice, pause, continue only if open or click rate passes | TOOL | `create_campaign.pause_rules` (pause_limit 1-50, wait_time minutes, pause_criteria open or click, pause_condition, pause_target); on a draft `update_campaign.pause_rules`; panel: Schedule > rules | All plans |
| Send a test email | Test mail, proof | Send the draft to chosen addresses | TOOL | `send_test_email` (campaign_id, recipients) | All plans |
| Test email groups | Saved groups | Reusable test recipient lists | TOOL (list); PANEL (create, edit) | `list_testmail_groups` (search), then pass the addresses to `send_test_email`; panel: Test mail modal > Saved groups | All plans |
| Audit before send | Pre-send check, checklist | Subject, sender, list, content and preheader checks | TOOL | `audit_campaign_draft` | All plans |
| Content check | Spam check, blacklist check, broken links, email size | Panel review checks 14 RBL zones and validates links; no spam score | PANEL | Wizard > Review; transactional has Email Review API | All plans |
| Blacklisted domain | Blacklist monitoring, RBL | A linked domain is on a blocklist | INFO | Help: "Blacklisted domain", "Blacklist monitoring tool" | All plans |
| Sender verification | Verify sender | Sender must be verified before send | INFO | `list_senders` shows status; verify in Settings > Sender Management | All plans |

## 4. Campaigns: scheduling and delivery modes

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Send now | Send immediately, instant, publish | Queue the campaign now (picked up within about 30 min) | TOOL, confirm first | `schedule_campaign` with no scheduled_at: first call without `confirm` returns the preview (recipient count, sender, subject); after the user says yes, call again with `confirm: true`. Avoid `create_campaign.is_publish=true`, which skips the preview | All plans |
| Schedule for later | Send later, scheduled send, schedule at | Send at one date and time in the account timezone | TOOL, confirm first | `schedule_campaign.scheduled_at` (YYYY-MM-DD HH:MM:SS): preview first without `confirm`, then `confirm: true` after the user agrees | All plans |
| Account timezone | Schedule timezone | Timezone used for scheduling; account-level | INFO | Settings > Account > Profile | All plans |
| Time-based campaign | Staggered send, deliver over time, time-based rules, send over days, "STO" as some clients say it | Spread one campaign over a date range, send days, hours per day and emails per hour | PANEL (no API) | Campaigns > draft > Schedule > Deliver over time (staggered) | All plans |
| Slice and send | Slots, campaign speed, add another slot | Split one day's send into up to 30 time slots | PANEL (no API) | Schedule > Staggered > slots | All plans |
| Optimized Sending | STO, Send Time Optimization, best time to send, smart send | Each contact gets the email at their best engagement hour within a 1 to 24 h window, with fallbacks | PANEL (no API) | Campaigns > draft > Schedule > Optimized sending | All plans |
| Deliver by timezone | Timezone delivery, recipient local time | Not available: there is no per-recipient timezone; the help article is outdated | INFO | Offer Schedule for later in the account timezone, or a time-based send | n/a |
| Resend to non-openers | Resend | Not found in the current panel | INFO | Suggest a segment "did not open campaign X" plus a new campaign | n/a |
| Pause campaign | Hold | Pause a sending campaign | PANEL (no API) | Campaigns > row actions > Pause | All plans |
| Resume campaign | Continue | Resume a paused campaign | PANEL (no API) | Campaigns > row actions > Resume | All plans |
| Abort campaign | Cancel, stop | Stop permanently; status Aborted | PANEL (no API) | Campaigns > row actions > Abort | All plans |
| Reschedule | Change send time | Set or change the send time of a draft | TOOL, confirm first | `update_campaign.scheduled_at`: setting a time publishes the campaign, so the first call returns a preview and only `confirm: true` applies it; a campaign already sending must be paused in the panel first | All plans |
| Duplicate campaign | Copy, clone | Copy a campaign to a new draft | PANEL (no API) | Campaigns > row actions > Duplicate | All plans |
| Delete campaign | Remove | Delete a draft or old campaign | PANEL (no API) | Campaigns > row actions > Delete | All plans |
| Campaign status | Draft, Scheduled, Queued, Sending, Paused, Aborted, Finished, Rescheduled, Failed | Lifecycle state | INFO | `list_campaigns`, `get_campaign` | All plans |
| Campaign queue | Two campaigns at once | Campaigns are queued and sent in order | INFO | Help FAQ "campaign queue when two run" | All plans |
| Autoresponder | Autoresponder series, follow-up campaign | No longer offered as a campaign type (legacy) | INFO | Build it as an advanced automation with a campaign-opened or clicked trigger plus wait and send-email nodes | All plans |

## 5. A/B split campaigns

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| A/B split campaign | A/B test, split test, A/B campaign | Two variants on one campaign, winner sent to the rest | PANEL (no API) | Campaigns > Create > A/B Split | All plans |
| Variant | Version A / B | Per-variant subject, sender, design | PANEL | A/B > Variants | All plans |
| Test size and wait | Split percentage, waiting time, rules | 1 to 50% per variant, wait minutes, hours or days | PANEL | A/B > Rules | All plans |
| Winner rule | Winning criteria, auto-pick winner | Winner chosen by open rate, then sent to the remainder | PANEL | A/B > Rules | All plans |
| A/B results | A/B report | Per-variant performance | PANEL | Reports > campaign > A/B | All plans |

## 6. Reports and analytics

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Campaign report | Campaign analytics, performance, overview | Delivered, opens, clicks, bounces, unsubscribes for one campaign | TOOL | `get_campaign`, `analyze_campaign` | All plans |
| Grade / score | Performance score, benchmark | Letter grade vs industry benchmarks | TOOL | `analyze_campaign`, `analyze_latest_campaigns` | All plans |
| Compare campaigns | Side by side, what changed | Metric-by-metric comparison with a winner | TOOL | `compare_campaigns` (campaign_ids) | All plans |
| Health check | Portfolio view, how are my campaigns doing | Quick grade across recent campaigns | TOOL | `campaign_health_dashboard` | All plans |
| Engagement funnel | Sent to delivered to opened to clicked | Drop-off at each stage | TOOL | `engagement_funnel` | All plans |
| Best practices | Recommendations, what should I improve | Data-driven recommendations from the account's own history | TOOL | `get_best_practices` | All plans |
| Open rate / click rate / CTR | Opens, clicks, click-to-open | Unique opens and unique clicks over delivered (sent minus hard bounces); click-to-open is unique clicks over unique opens. Campaigns with under 100 delivered are not graded | INFO | From the tools above | All plans |
| Unique vs total opens | Unique opens | Deduplicated per contact vs all events | INFO | `get_campaign` | All plans |
| Domain report | Deliverability by provider, Gmail vs Outlook | Per-recipient-domain performance | TOOL | `get_campaign_domain_report` | All plans |
| Opens list / clicks list / unsubscribes list | Who opened, who clicked, recipient activity | Per-contact engagement lists | TOOL | `get_campaign_opens`, `get_campaign_clicks`, `get_campaign_unsubscribes` (campaign_id, page, limit 1-100, date_from, date_to); unsubscribes include the reason (Abuse = reported spam) | All plans |
| Click map | Heatmap, link report | Clicks overlaid on the email | PANEL | Reports > campaign > Click Map | All plans |
| Location report | Geo map, country report | Engagement by country | PANEL | Reports > campaign > Location | All plans |
| Browser / device report | Device status | Engagement by browser and device | PANEL | Reports > campaign > Browser/Devices | All plans |
| Timestamp / timeline | Opens by time, engagement over time | Hourly or daily engagement curve | PANEL | Reports > campaign > Opens/Clicks | All plans |
| Unsubscribe reasons | Departure reasons, reason analytics | Why people unsubscribed | PANEL | Reports > campaign > Unsubscribes & Complaints; Settings > Unsubscribe Page | All plans |
| Spam complaints / abuse | Complaints, FBL, marked as spam | Recipients who reported the email | INFO | `get_campaign`; help: "marked as spam" | All plans |
| Bounce types | Hard bounce, soft bounce, transient | Hard = invalid address (suppressed); soft or transient = retry | INFO | `get_campaign`; help: "Bounce types" | All plans |
| Inbox placement report | Inbox Tracker results, inbox rate, spam folder rate | Inbox vs spam vs promotions by provider from seed addresses | TOOL | `get_inbox_tracking` (date_from, date_to, campaign_id, domain) | Add-on |
| Optimized Sending report | STO report | Results of optimized sending | PANEL | Reports > campaign > Optimized Sending | All plans |
| Export report | Download report, CSV export | Export analytics | PANEL | Reports > campaign > Export | All plans |
| Advanced reports | Multi-campaign report, campaign comparison page | Cross-campaign table with export | PANEL | Reports > Campaigns | All plans |
| Automation report | Automation stats, delivery report, campaign-wise report | Per-automation sends and activity | PANEL | Reports > Automations | All plans |
| Column visibility | Customize columns | Choose metrics shown in the campaign list | PANEL | Campaigns > columns | All plans |

## 7. Audience: contacts and lists

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Contact | Subscriber, audience member, lead, email ID | One person with an email in a list | TOOL | `get_contact`, `create_contact`, `update_contact`, `delete_contact` | All plans |
| List | Contact list, audience list, recipient list, mailing list | Container of contacts; a contact can be in several lists | TOOL | `list_contact_lists`, `get_list_details`, `create_list` (name, list_type), `update_list`, `delete_list` | All plans |
| Add a contact | Add subscriber, create contact | Create one contact in a list | TOOL, confirm first | `create_contact` (email, list_id, names, phone, company, city, state, country, job_title, department, tags, custom_fields) | All plans |
| Upsert | Create or update, add if missing | Update by email or create | TOOL | `upsert_contact` (email, list_id, first_name, last_name) | All plans |
| Bulk add | Batch, import list, add many | Up to 50 contacts per call | TOOL | `batch_create_contacts` (list_id, contacts[]); larger files: panel import | All plans |
| Import wizard | Add Contacts, upload CSV/XLSX, copy and paste, map columns | File import with column mapping, tags and contact type | PANEL | Audience > Add Contacts | All plans |
| Import summary | Import report, import log | Result of an import job | PANEL | Audience > Add Contacts > summary; Settings > Logs | All plans |
| Contact type on import | Import as active / unsubscribed / bounced / suppressed | Status assigned to imported rows | PANEL | Import > map | All plans |
| Export audience | Export contacts, download list | Emailed, expiring download link | PANEL | Audience > Contacts or Lists > Export | All plans |
| Contact details | Contact profile, activity log | Fields, tags, lists and event timeline for one contact | TOOL (fields); PANEL (timeline) | `get_contact`; Audience > Contacts > contact | All plans |
| Contact status | Active, Unverified, Unsubscribed, Bounced, Spam complaint, Suppressed | Sendability state | INFO | `get_contact`, `list_contacts` | All plans |
| Unsubscribe / resubscribe | Opt out, opt in again, change contact status | Change whether Marketing Platform campaigns can reach a contact | TOOL, confirm first | `update_contact` with `fields.contact_type` (active, unsubscribe, suppressed, bounce, abuse, spam complaints): the first call returns a preview, `confirm: true` applies it; bulk changes: panel, Audience > Contacts > select | All plans |
| Bulk actions | Cleanup, bulk delete, bulk tag, move list, bulk unsubscribe | Mass changes from the contact grid | PANEL | Audience > Contacts > select > Actions | All plans |
| Delete contact | Remove subscriber | Delete one contact; irreversible | TOOL, confirm first | `delete_contact` (contact_id): preview first, then `confirm: true` | All plans |
| Double opt-in | DOI, confirmed opt-in, confirmation email | Contact confirms by email before becoming active | PANEL | Audience > Lists > list > DOI; Import; Webform settings | All plans (watermark on Free) |
| Unknown contacts | Unverified contacts | Contacts without consent or verification | INFO | Help: "Unknown contacts" | All plans |
| Contact rating | Contact score, star rating, lead score, engagement rating | Engagement rating per contact; owner-controlled scoring | INFO | Visible in `get_contact` where returned; Audience > Contacts column | All plans |
| Global openers | Cross-account openers | Contacts seen opening across the platform | INFO | Segment filter; per-tenant flag | All plans |
| Verify a list | Clean a list, email verification, deliverability score, risk level | Run the Email Verifier on a list or contacts | WRAP | Verifier API; panel: Audience > Lists > Verify | Add-on credits |
| Duplicate contacts | Same contact in multiple lists, allow duplicates | Policy for a contact in several lists | INFO | Settings > Preferences | All plans |
| Open tracking consent | Consent, lawful basis, when consent unknown | Whether opens are tracked for a contact | PANEL | Settings > Preferences; Import | All plans |
| Audience summary | Contact summary | Cross-list totals, engagement and verification | PANEL | Audience > Summary | All plans |
| Suppression list (marketing) | Suppressed contacts, do-not-mail for campaigns | Stop Marketing Platform campaigns reaching an address | TOOL, confirm first | `update_contact` with `fields.contact_type: suppressed` or `unsubscribe` (preview, then `confirm: true`); in bulk, import with Contact type Suppressed in the panel. `add_suppressions` does NOT affect Marketing Platform campaigns: it is the API Platform list | All plans |
| Preference center | Subscription preferences, manage preferences | Hosted page where contacts choose lists | PANEL | Settings > Account > Preference Center | All plans |
| Unsubscribe page | Unsub page, unsubscribe link, departure survey | Branded unsubscribe and success page | PANEL | Settings > Account > Unsubscribe Page | All plans |

## 8. Custom fields, tags and segments

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Custom field | Contact property, custom property, attribute, merge field | Per-account contact field (text, number, date, textarea) | TOOL | `list_custom_fields`; `create_custom_field` (name 3-30, type, description); `update_custom_field` (field_id, name, description; type is fixed); `delete_custom_field` (field_id, confirm; fails if used in a webform) | All plans |
| Tag | Contact tag, label | Free-form label on contacts | TOOL | `list_tags`, `create_tag`, assign via `create_contact.tags`; `update_tag` (tag_id, name); `delete_tag` (tag_id, confirm); `remove_tag_from_contact` (contact ID or email, tag_ids and/or tag_names, confirm) | All plans |
| Segment | Smart list, dynamic list, filter, saved filter | Saved condition set over contacts, recomputed on refresh | TOOL | `list_segments`; `create_segment` (name, match all/any, lists by name, rules with filter_type contact_details, contact_property, tag or activity); `update_segment` (segment_id plus the full name, match and rules: it replaces everything) | All plans |
| Segment conditions | Filters, rules, criteria | Contact properties, contact details, activity, campaign activity, sender, automation, conversion, tags | INFO | Help: "Segment conditions"; max filters per segment set by server | All plans |
| Predefined segments | Engaged, dormant, never opened, openers, non-openers | Pre-filled segment templates | PANEL | Audience > Segments > Predefined | All plans |
| Refresh segment | Recalculate, segment not updating | Recompute the segment count | PANEL | Audience > Segments > row > Refresh | All plans |
| Segment analytics | Segment summary, composition | Charts for a segment | PANEL | Audience > Segments > segment | All plans |
| Save filter as segment | Save filter | Turn a contact-grid filter into a segment | PANEL | Audience > Contacts > filter > Save as segment | All plans |

## 9. Automation

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Automation | Workflow, journey, drip, sequence, email automation, autoresponder flow | Triggered multi-step email flow | TOOL (read); WRAP (create) | `get_automation` (automation_id, node_type); API `Create Automation`; panel: Automations | All plans |
| Classic automation | Basic workflow editor, 3-step automation | Trigger, campaigns with wait rules, publish | PANEL | Automations (classic) | All plans |
| Advanced automation | Flow builder, visual workflow, canvas, 14 node types | Node-based builder | PANEL | Automations > Create advanced | All plans |
| Bulk automation | Bulk send automation | Automation at bulk scale | PANEL | Automations | All plans |
| Trigger | Starting trigger, entry condition | Joins list, webform submit, API add, property change, tag added, campaign opened or clicked | INFO | Help: "Adding starting triggers" | All plans |
| Wait / delay | Wait for period, wait till time, wait for trigger | Delay nodes | INFO | Help: "Set delays" | All plans |
| Condition | If/Else, branch, filter condition | Branch on contact or activity filters | INFO | Help: "Add conditions" | All plans |
| Split | Random split, A/B path | Random branch | INFO | Flow > Split node | All plans |
| Go to | Jump | Jump to another node | INFO | Flow > Go To | All plans |
| Action nodes | Send email, update contact, add or remove tag, add to or remove from list, delete contact | What the flow does | INFO | Help: "Add actions" | All plans |
| Enrollment | Enrolled contacts, status updates, re-entry | Contacts in the flow and whether they may re-enter | INFO | Help: "Overview of automation workflows"; "Allow contacts to re-enter" | All plans |
| List automations | All automations, find by name | See automations with status | PANEL (no API) | Automations | All plans |
| Activate / deactivate automation | Enable, disable, pause automation, inactivate | Lifecycle control | PANEL (no API) | Automations > row actions | All plans |
| Duplicate / rename / delete automation | Copy workflow | Housekeeping | PANEL (no API) | Automations > row actions | All plans |
| Automation templates | Predefined automations | Prebuilt flows to clone | PANEL | Automations > templates | All plans |
| Test automation | Dry run | Not available | INFO | Suggest a small test list and activating the automation | n/a |
| Welcome automation | Welcome series, onboarding flow | Common trigger-on-join flow | INFO | Help FAQ "welcome automation trigger via API/manual/import" | All plans |

## 10. Webforms

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Webform | Web form, signup form, form builder, subscription form, lead form | Form that adds contacts to a list | TOOL (list only) | `list_webforms`; panel: Audience > Webforms | All plans |
| Form types | Embedded inline, standalone page, pop-up, dropdown banner, slide-in, floating bar, fullscreen welcome mat | Display formats (8) | PANEL | Webform > New > type | All plans |
| Create form | From scratch, from template, AI form generator, import from URL | Ways to create | PANEL (no API) | Webform > New | All plans |
| Form fields | NPS, star rating, picklist, image choice, multi-step | Field types | PANEL | Editor > Form tab | All plans |
| Display rules | Triggers, targeting, frequency, exit intent, scroll | When and where a popup shows | PANEL | Editor > Rules | All plans |
| Form settings | Target list, tags, double opt-in, duplicates, redirect, thank-you message, notification | Submission behaviour | PANEL | Editor > Settings | All plans |
| Captcha | Turnstile, reCAPTCHA | Bot protection | PANEL | Editor > Settings | All plans |
| Embed code / share link / hosted page | Publish form, form on website, share.mailercloud.com | Pinned embed snippet or hosted URL | PANEL | Webform > Publish | All plans |
| Form report | Submissions, form analytics, views | Views, submits, new contacts, CSV | PANEL | Webform > Report | All plans |
| WordPress forms | WP plugin, Contact Form 7, WPForms, Elementor sync | Connect WordPress forms and users | PANEL | Settings > Plugins > WordPress; help: WordPress plugin | All plans |

## 11. Account settings and deliverability setup

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Sender management | Senders, sender IDs, from addresses, verify sender | Add and verify sending addresses | TOOL (list); PANEL (add, verify) | `list_senders`; Settings > Account > Sender Management | All plans |
| Reply IDs | Reply-to addresses | Add and verify reply-to addresses | TOOL (list); PANEL (add) | `list_reply_emails`; Settings > Account > Reply IDs | All plans |
| Domain authentication | Domain auth, verify domain, custom domain, sending domain, SPF, DKIM, DMARC, CAA | Prove domain ownership and authenticate mail | TOOL | `add_domain` (domain; returns the DNS records to publish), `list_domains` (page, limit 10-100, search, status verified, not_verified or failed), `get_domain` (domain_id; per-record status and fix); panel: Settings > Domains | All plans |
| One-click DNS connect | Domain Connect, Entri, auto DNS | Publish DNS records automatically | PANEL | Settings > Domains > Add > Connect | All plans |
| DMARC reconfigure | Fix DMARC | Repair a DMARC record | PANEL | Settings > Domains > Reconfigure DMARC | All plans |
| Share domain | Domain sharing | Share an authenticated domain with another account | PANEL | Settings > Domains | All plans |
| Tracking domain | Custom tracking domain, branded links | Not available to clients | INFO | Say it is not offered; domain authentication covers SPF, DKIM and DMARC | n/a |
| Dedicated IP | Dedicated sending IP, own IP | Private IP with warm-up | UPGRADE | Add-on, $50 per IP per month; Settings > Billing & Usage > Add-ons; API: Pro and Platform plans | Add-on |
| IP warm-up | Warming up | Gradual volume ramp on a new IP; statuses Setting up, Warming up, Active | INFO | Dedicated IP add-on page | Add-on |
| Blacklist monitoring | RBL monitor | Watches sending domains and IPs on blocklists | PANEL | Help: "Blacklist monitoring tool" | All plans |
| Timezone (account) | Profile timezone | Used for scheduling | PANEL | Settings > Account > Profile | All plans |
| Profile | Company details, billing address, tax ID, avatar | Account identity | PANEL | Settings > Account > Profile | All plans |
| Preferences | Account defaults, frequency cap window, duplicate policy, open-tracking default | Account-level defaults | PANEL | Settings > Account > Preferences | All plans |
| Users | Team members, multi-user, sub-users, invite user | Add people to the account | PANEL (no API) | Settings > Account > Users | All plans (SAML SSO: Enterprise) |
| Roles and permissions | RBAC, Admin, Manager, Analyst, Viewer | Permission levels per user | PANEL | Settings > Account > Users > Role | All plans |
| Sub-accounts | Client accounts, agency accounts | Separate accounts per client | PANEL | Help: "Sub-accounts / client-specific accounts" | All plans |
| Two-factor authentication | 2FA, TOTP, authenticator app, backup codes | Second login factor | PANEL | Settings > Account > Security | All plans |
| Secure team | Enforce 2FA | Require 2FA for all users | PANEL | Settings > Security | All plans |
| SSO | Google sign-in, Microsoft sign-in, SAML | Federated login | PANEL | Login page; SAML: Enterprise | Google/Microsoft Free+; SAML Enterprise |
| Audit log | Activity log, account history | Who changed what | PANEL | Settings > Account > Audit Log | All plans |
| API key | API token, generate API key | Key for the marketing REST API | PANEL (never return a key in chat) | Settings > Integrations > API Keys | All plans |
| Client logs | Contact logs, import logs | Per-job logs | PANEL | Settings > Logs | All plans |
| Conversion pixel | Tracking pixel, image/iframe/JavaScript/postback pixel | Conversion event capture | PANEL | Settings > Account > Conversion Pixel | All plans |
| Campaign tag settings | Manage campaign tags | CRUD for campaign tags | PANEL | Settings > Account > Campaign tag | All plans |
| Change registered email | Account email | Change the login email | PANEL | Settings > Account > Profile | All plans |

## 12. Webhooks and integrations

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Webhook | Callback, event notification, HTTP push | POST on Marketing Platform events: campaign_sent, campaign_opened, campaign_clicked, unsubscribe, campaign_error, spam, hard_bounce | TOOL | `list_webhooks`, `get_webhook`, `create_webhook` (name, url, events; only these event names are accepted), `update_webhook` (name, url, events; unchanged fields are kept), `delete_webhook`, `toggle_webhook`, `test_webhook`. Marketing webhooks only: transactional and Email API webhooks are in the panel | All plans |
| Test webhook | Fire test event | Send a sample payload | TOOL | `test_webhook` (webhook_id, dry_run); sends a real request to the customer's URL | All plans |
| Webhook signing | Payload signing, HMAC, signing secret | Signature header on webhook deliveries | PANEL | Account > Webhooks (marketing) or Transactional > Settings > Payload signing; not available through the connector | All plans |
| Zapier / Make / Pabbly / Integrately / Albato / KonnectzIT | Automation platforms, no-code integration | Connect Mailercloud to other apps | PANEL | Settings > Plugins; help: Zapier integration | All plans |
| HubSpot / Intercom / Salesforce / Pipedrive | CRM sync | Contact sync | PANEL | Settings > Plugins | All plans |
| Shopify / WooCommerce / Magento / BigCommerce | E-commerce integration | Store customer sync | PANEL | Settings > Plugins | All plans |
| WordPress plugin | WP plugin | Forms and user sync from WordPress | PANEL | wordpress.org plugin; Settings > Plugins | All plans |
| Meta custom audiences | Facebook audiences sync | Sync lists to Meta ad accounts | PANEL | Settings > Plugins > Meta | All plans |
| WebEngage / CleverTap / MoEngage / PoperAI | CDP integration | Use Mailercloud as the email channel | PANEL | Transactional > Integrations; Settings > Plugins | API plan |
| Stripo / Tabular export | Template import from editors | Bring templates from external editors | PANEL | Templates > Import | All plans |
| Google Sheets / Google Forms / Typeform | Form sources | Add contacts from forms | PANEL | Settings > Plugins | All plans |

## 13. Transactional (API Platform)

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Send transactional email | Email API send, API email, triggered email | One-to-one application email | TOOL, confirm first | `send_transactional_email` (from, from_name, subject, to, cc, bcc, text, html, amp_html, reply_to, attachments, version 1.0 or 2.0): first call without `confirm` returns a preview; send with `confirm: true` only after the user agrees | API plan |
| Personalized send | Send Personalized Email, mail merge, bulk send API | One call, many recipients with merge data | TOOL | `send_personalized_email` (from, subject, to with per-recipient merge_vars object, html or text; combined merge_vars at most 100 KB); two-step `confirm` like `send_transactional_email` | API plan |
| SMTP relay | SMTP credentials, SMTP password, port | Send via SMTP instead of REST | PANEL | Transactional > Connection Guide | API plan |
| Connection guide | Setup, credentials, API key (transactional) | Credentials and test send | PANEL | Transactional > Connection Guide | API plan |
| Account approval | Request approval, account review | Review gate before sending | PANEL | Transactional > Request approval | API plan |
| Email Review API | Pre-send review, content review | Checks links, blacklists, AMP errors, size for transactional content | WRAP | Email Review API (transactional only); help: "Email Review API help guide" | API plan |
| Activity log | Activities, email logs, message status, SMTP response | Per-message delivery status with preview | PANEL | Transactional > Activities | API plan (retention 7/30 days by plan) |
| Content storage | Store content | Keep message bodies for preview | PANEL | Transactional > Settings | API plan |
| Suppression list (API Platform) | Suppressed recipients, blocked addresses, Email API suppression | Addresses rejected by the Email API and SMTP relay; reasons hard bounce, spam complaint, manual. Scope `global` = all API Platform mail, `marketing` = promotional Email API messages (campaignType PROMOTIONAL), `transactional` = transactional messages. It never affects Marketing Platform campaigns | TOOL | `list_suppressions`, `add_suppressions` (dry_run first), `reactivate_suppression` (spam-complaint entries cannot be reactivated) | API plan |
| Transactional webhooks | Event callbacks, delivery events, Email API webhooks | Delivery, open, click, bounce events for API Platform mail, incl. custom data | PANEL | Transactional > Webhooks; the connector's webhook tools manage Marketing Platform webhooks only | API plan |
| Custom data | Metadata, custom fields on a message, X-MC-Custom | Sender-supplied data on a transactional message, returned in webhooks and the activity log | INFO | Email API reference at apidoc.mailercloud.com | API plan |
| IP allowlist | IP whitelist | Restrict API and SMTP use to listed IPs | PANEL | Transactional > Settings > IP Allowlist | API plan |
| Open tracking (transactional) | Per-domain tracking | Track opens on transactional mail | PANEL | Transactional > Settings | API plan |
| Inbox Tracker (API) | Inbox rate, placement for API mail | Seed-based placement for Email API streams | PANEL | Transactional > Inbox tracker. `get_inbox_tracking` covers Marketing Platform campaigns only | Add-on |
| Usage | Quota, volume, emails used | Consumption against the API plan | PANEL | Transactional > Usage | API plan |
| Dedicated IP (API) | Own IP for API mail | Private IP | UPGRADE | Add-on; API Pro and Platform only | Add-on |
| Inbound parsing | Inbound email, receive email | Not yet available | INFO | Pricing page says "coming soon" | n/a |

## 14. Email Verifier

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Quick verify | Single email verification | Up to 50 pasted addresses | WRAP | Verifier API `Verify Emails`; panel: Email Verifier > Single | Credits |
| Bulk verify | List verification, upload file | Verify a whole file of addresses | PANEL | Email Verifier > List Verification (the Verifier API takes CSV/TXT up to 10 MB); not yet available through the connector | Credits |
| Results | Deliverable, invalid, risky, unknown, safe to send, delivery score 0 to 100 | Verification outcome per address | INFO | Download results in the panel | Credits |
| Credits | Verifier credits, top-up | One credit per address; packs never expire; 100 free for marketing subscribers | UPGRADE | Email Verifier > Pricing | Credits from $5 per 1K |
| Verifier API key | Verifier API | Separate keys for the verifier | PANEL | Email Verifier > API Keys | Credits |

## 15. Plans, billing and add-ons

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Free plan | Free tier | 1,000 contacts, 12,000 emails per month; every feature is available | INFO | `get_account_overview` | Free |
| Premium plan | Paid plan, Pro | From $10 per month for 5,000 contacts; unlimited emails (fair use), higher contact limits, priority chat support | UPGRADE | Settings > Billing & Usage > Change plan | Premium |
| Enterprise plan | Custom plan, request a quote | Custom contacts, unlimited automations, SAML SSO, account manager | UPGRADE | Settings > Billing & Usage > Request a quote | Enterprise |
| Email API plans | Basic, Pro, Platform | $10, $100, $450 per month; 50K, 500K, 2.5M emails; 14-day trial | INFO | Transactional > Change plan | API plan |
| Add-ons | Power-ups | Dedicated IP $50 per IP per month; Inbox Tracker $120 per month; Email Marketing as a Service $350 per month | UPGRADE | Settings > Billing & Usage > Add-ons | Add-on |
| Email Marketing as a Service | EMaaS, managed service, done-for-you | Mailercloud team runs campaigns for the client | INFO | Help: "Email Marketing as a Service" | Add-on |
| Change plan | Upgrade, downgrade, switch plan, annual billing | Stripe-hosted checkout or portal; 30% off annual | PANEL (never take payment in chat) | Settings > Billing & Usage > Change plan | All plans |
| Reduce contacts | Fit a plan | Lower contact count before downgrade | PANEL | Change plan > Reduce contacts | All plans |
| Coupon | Promo code, discount | Applied at checkout | PANEL | Change plan > Coupon | All plans |
| Payment methods | Cards, saved card | Managed in the Stripe portal | PANEL | Settings > Payment Methods | Premium |
| Invoices | Billing history, receipts | Download invoices | PANEL | Settings > Invoices | Premium |
| Cancel / resume subscription | Stop plan, restart subscription | Stripe Customer Portal | PANEL | Settings > Billing & Usage > Manage subscription | Premium |
| Usage and limits | Contacts used, emails sent this month, quota | Consumption against the plan | TOOL (contacts); PANEL (emails, quota) | `get_account_overview` gives plan and contact counts; emails sent, API quota and add-on status: Settings > Billing & Usage | All plans |
| Fair use | Unlimited emails | Unlimited on Premium subject to fair use | INFO | Help: "Plan and pricing FAQs" | Premium |

## 16. Analysis-only terms the assistant should recognise

| Term | Meaning | Action |
| --- | --- | --- |
| "How did my last campaign do" | Performance summary | `analyze_campaign` on the latest finished campaign |
| "What should I fix first" | Prioritised recommendations | `get_best_practices` or `analyze_latest_campaigns` |
| "Why did opens drop" | Diagnosis | `compare_campaigns` plus `get_campaign_domain_report` |
| "Is my email landing in spam" | Placement | `get_inbox_tracking` if the add-on is active, else `get_campaign_domain_report` and bounce or complaint rates |
| "Clean my list" | Hygiene | Suggest a segment of non-openers, suppression of hard bounces, and the Email Verifier; or the `clean_my_list` prompt |
| "Warm up" | IP or domain warm-up | Explain gradual volume ramp; Dedicated IP add-on handles IP warm-up |
| "Sender reputation" | Deliverability standing | Bounce, complaint and placement data from the tools above |
| "Industry benchmark" | Comparison baseline | Built into `analyze_campaign` scoring |

## 17. Guided workflows (MCP prompts)

Clients that support MCP prompts show these as ready-made starting points. Each one tells the assistant which tools to chain and which guardrails apply.

| Term | Also called | Meaning | Outcome | Action | Plan |
| --- | --- | --- | --- | --- | --- |
| Getting started | What can you do, what can Mailercloud do, help me start | Overview of every product and add-on, what works from chat and what is done in the app, then first tasks | TOOL | Prompt `getting_started`: `get_account_overview` (its product overview section), then three first tasks for the product the user picks | All plans |
| Plan a campaign | Campaign plan, help me send a campaign | Audience, content, draft, audit and test send, ready to schedule | TOOL | Prompt `plan_campaign` (goal, audience): `list_contact_lists`, `list_segments`, `create_campaign` as draft, `audit_campaign_draft`, `send_test_email` with `dry_run: true` first, then `schedule_campaign` two-step | All plans |
| Audit a draft | Check my draft, pre-send review | Prioritised fix list for one draft | TOOL | Prompt `audit_draft` (campaign_id): `get_campaign`, `audit_campaign_draft`, `list_senders` | All plans |
| Monthly review | Monthly report, how did this month go | Recent performance with three actions | TOOL | Prompt `monthly_review` (count, default 10): `analyze_latest_campaigns`, `campaign_health_dashboard`, `compare_campaigns`, `get_best_practices` | All plans |
| Clean my list | List hygiene, list cleanup | Health of one list and safe clean-up options | TOOL | Prompt `clean_my_list` (list_id): `get_list_details`, non-openers segment in the panel, `add_suppressions` with `dry_run: true` first, Email Verifier (paid credits); never delete or suppress without explicit confirmation | All plans |
| Report a gap | Feature request, unsupported task | Log an ask the connector cannot fulfil | TOOL | `report_unsupported_task` (request, term, outcome, panel_path); no Mailercloud API call | All plans |

---

## Maintenance

- A term is added or changed here first, then the three renderings are regenerated (server `Instructions` via `go:embed`, plugin skill section, help article table).
- When a tool is added, change the row's Outcome from WRAP or PANEL to TOOL in the same PR.
- `report_unsupported_task` logs become issues against this file; unknown terms become rows.
- Plan gating confirmed by product 2026-10-16: every marketing feature is available on every plan, including A/B split, AI subject lines, AMP (access request), Optimized Sending and advanced automation. Plans differ only in contact and email limits and support level; SAML SSO is Enterprise. Paid extras are the add-ons: Inbox Tracker, Dedicated IP, Email Marketing as a Service, Verifier credits. The public pricing page still lists some of these under Premium; that is a listing choice, not a restriction.
