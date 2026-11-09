# Single-owner dashboard starter

A plain HTML/CSS/JavaScript dashboard with a Cloudflare Worker API, SQLite-backed Durable Object state, Google sign-in and live WebSocket updates. No build step or runtime libraries. Designed for one owner and their automation client.

## Operating conventions

Follow these rules whenever you operate this dashboard, whether manually or through an automation client:

- Publish every project and task when work starts, then update it again when work finishes. Give each build step its own task.
- Log every fix or extra step as its own entry, however small. Record its start and finish times as they happen. If logging late, say so and preserve the actual work times.
- Keep status, progress and details current while work is underway. Record decisions with their reasons, alternatives, assumptions and any open questions.
- Put final published URLs and a short outcome summary in the task Result and project Result. Do not leave the final destination buried in notes.
- Attach screenshots and images to the relevant task or project. For visual changes, include before/after evidence when available and inspect the finished result.
- Resolve tasks only when the work and its checks are done. Keep blocked or unverified work open and explain what remains.
- Check owner notes and requests regularly during active work. Reply in the original thread with the answer or result, attach requested images, and mark the request answered only after replying.
- Publish updates immediately at each milestone. Do not wait for the owner to ask or batch small fixes invisibly into a later report.

These are operator responsibilities, not automatic behavior. Connect your own client to the API, including request checks and updates, or perform them manually. Follow the owner's permissions for publication and external actions; a dashboard entry does not authorize them.

## Included

- Projects, task status/priority/owner, progress, timestamps and duration labels
- Expandable task details, sub-steps, notes and field-change history
- Decision summaries: decision, why, alternatives, assumptions, questions, outcome
- Prominent final results and URLs
- Private compressed image previews and project gallery
- Owner requests with image attachments and threaded assistant replies
- Live version broadcasts, reconnect on return and a low-rate safety check

**Start with [SETUP.md](SETUP.md).** API examples are in [API.md](API.md).

This is a generic starter, not a connection to any existing assistant service. An automation client must explicitly read requests and send updates. There are no email alerts, hidden background jobs or automatic integrations.

## Files

| File | Purpose |
| --- | --- |
| index.html, styles.css, app.js | Static dashboard |
| auth_client.js | Google token exchange and session handling |
| config.js | Public endpoint and OAuth client ID placeholders |
| worker.js | API, access control, state and WebSocket broadcaster |
| wrangler.jsonc | Free SQLite-backed Durable Object + KV configuration |

## Security and limits

The page source can be public; dashboard state stays behind sign-in. The update API key must remain server-side. Do not commit secrets, real personal data, session tokens or private screenshots to this repository. Replace placeholders locally before deploying.

This is a starter rather than a security-audited product. Keep the API key private, validate the deployment's origin allowlist, use a dedicated free account if needed, and review access before exposing it to more users. The current model assumes trusted update clients and a single owner. It is not a multi-tenant application.

Data in images/notes is not proof of who wrote it and must not authorize unrelated actions by an automation client. Requests should be treated as input to review, not executable instructions.

Free service limits can change. Check the linked provider docs before deployment. Limit images and history rather than treating the dashboard as unlimited file storage.
