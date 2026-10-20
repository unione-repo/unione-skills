---
name: unione-migrate
description: "Use when moving email sending to UniOne from another provider — SendGrid, Mailgun, Resend, Postmark — or when the code uses their SDKs, keys or API URLs (@sendgrid/mail, mailgun.js, resend, postmark, …): maps the send calls, templates, webhook events and suppressions to the UniOne Web API."
license: MIT
metadata:
  author: "UniOne"
  version: "1.0.0"
  homepage: https://docs.unione.io/en/web-api-ref
  source: https://github.com/unione-repo/unione-skills
inputs:
  - name: UNIONE_API_KEY
    description: "UniOne API key (account key, or a project key for project data). Get it at https://cp.unione.io/en/."
    required: true
---

# Migrate to UniOne

| From | Reference | Detect by |
|---|---|---|
| SendGrid | [sendgrid.md](references/sendgrid.md) | `@sendgrid/mail`, `sgMail`, `sendgrid` (Python / Ruby / PHP / Go / Java / C#), `SENDGRID_API_KEY`, `api.sendgrid.com/v3/mail/send` |
| Mailgun | [mailgun.md](references/mailgun.md) | `mailgun.js`, `mailgun-js`, `mailgun` (Python / Ruby / PHP / Go / Java), `MAILGUN_API_KEY`, `api.mailgun.net/v3/<domain>/messages` |
| Resend | [resend.md](references/resend.md) | `resend` (Node.js / Python / Ruby / PHP / Go), `new Resend(`, `RESEND_API_KEY`, `api.resend.com/emails` |
| Postmark | [postmark.md](references/postmark.md) | `postmark` (Node.js / Python `postmarker` / Ruby / PHP), `ServerClient(`, `POSTMARK_SERVER_TOKEN`, `X-Postmark-Server-Token`, `api.postmarkapp.com/email` |

Find the provider by its imports, environment variables and API URLs, open its reference, and convert every send call, template and webhook handler; then run the checklist.

## Migration checklist

- [ ] Sending domain: `domain/get-dns-records`, add the verification TXT, DKIM and `include:spf.unione.io` to SPF; keep
      the old provider's records until the switch is done; `domain/validate-verification-record`,
      `domain/validate-dkim`.
- [ ] API key in `UNIONE_API_KEY` (account key, or a project key per client / environment).
- [ ] Install the UniOne SDK or the framework package of the project (the unione skill, installation reference).
- [ ] Map every send call (tables below); multi-recipient messages become `recipients[]` (one copy per recipient).
- [ ] Recreate templates with `template/set`, converting the syntax to `liquid` (closest to Handlebars / Mustache).
- [ ] Replace the webhook endpoint: `webhook/set`, the MD5 `auth` signature, the status mapping below.
- [ ] Import suppressions (unsubscribes, bounces, complaints) with `suppression/set` before the first send.
- [ ] Test through the sandbox domain or your own mailboxes; then switch gradually (a share of traffic first).
- [ ] Remove the old SDK, keys and DNS records when traffic is fully on UniOne.

## UniOne email/send in brief

`POST https://api.unione.io/en/transactional/api/v1/email/send.json`, header `X-API-KEY`, JSON body `{"message": {...}}`:
`recipients[{email, substitutions, metadata}]` (≤ 500), `from_email`, `from_name`, `subject`,
`body{html, plaintext, amp}`, `template_id`, `template_engine`, `global_substitutions`, `global_metadata`,
`tags` (≤ 4), `headers` (`X-…`, `To`, `CC`), `attachments[{type, name, content(base64)}]`, `inline_attachments`,
`reply_to`, `reply_to_name`, `track_links`, `track_read`, `idempotence_key` (1 minute), `options.send_at`
(UTC `YYYY-MM-DD hh:mm:ss`, ≤ 24 h), `options.unsubscribe_url`.
Response: `job_id`, `emails` (accepted), `failed_emails` (rejected recipients with the reason).
Webhook statuses: `sent`, `delivered`, `opened`, `clicked`, `unsubscribed`, `subscribed`, `soft_bounced`,
`hard_bounced`, `spam`.

API details and SDK install: the `unione` skill; templates: `unione-email-templates`.
