# Setup

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

## 1. Prerequisites

Use accounts you control for GitHub, Cloudflare and Google Cloud. No paid plan is required for the components in this starter. Google OAuth setup may require account verification or organization approval. Do not bypass those checks.

You need Node.js and Wrangler for backend deployment. The static frontend needs no package install or build.

## 2. Create a public code repository

Copy these template files into a new repository. Keep live data and secrets out. Enable GitHub Pages from the main branch/root after the first commit. Record the URL GitHub gives you. For a project repository it usually includes a path; use the actual URL, not an assumed one.

## 3. Google sign-in

1. In Google Cloud Console, create a project and configure its OAuth consent/branding screen.
2. Create an OAuth client of type **Web application**.
3. Add your GitHub Pages origin to Authorized JavaScript origins, such as `https://YOUR_GITHUB_USERNAME.github.io`. An origin has no project path.
4. Set the public client ID in `config.js` and `wrangler.jsonc` (`CLIENT_ID`). These must match.
5. Set `ALLOWED_EMAILS` to a comma-separated list of the specific owner accounts permitted to view the dashboard. Only add accounts with the owner's approval.
6. Meet Google's testing, verification and publishing requirements shown in your console. This starter does not bypass them.

The browser sends a Google ID token to the Worker. The Worker checks it with Google's tokeninfo endpoint, verifies the audience and verified-email claim, checks the allowlist, then issues a 12-hour opaque session. No Google client secret is used by the frontend.

## 4. Cloudflare backend

1. Authenticate Wrangler with your own Cloudflare account: `npx wrangler login`.
2. Create a KV namespace: `npx wrangler kv namespace create KV`.
3. Put its returned namespace ID in `wrangler.jsonc`.
4. In `worker.js`, replace the `ALLOWED_ORIGINS` placeholder with the exact Pages origin. Remove localhost if you do not use local development.
5. Set a strong update key using `npx wrangler secret put API_KEY`. Enter it only at the secret prompt, never into source or public config.
6. Deploy using a current Wrangler version: `npx wrangler deploy`.
7. Put the returned HTTPS Worker endpoint in `config.js`. Commit only the public frontend configuration, never the update key.

The SQLite Durable Object holds projects/tasks/notes/images/version. KV holds sign-in sessions only. Data is split into storage chunks because a Durable Object key-value entry has a per-value size limit. Keep the existing namespace and migration history when updating, or you can accidentally create an empty database. Never delete a Durable Object namespace to fix a deployment.

## 5. Verify before using real data

- Open the page signed out. Confirm that private data is not visible.
- Sign in with an approved account. Test a non-approved account only if you control it.
- Send one project and task through the API, then resolve it. Verify the rendered page without reload.
- Submit a task request with an image. Read it via `listNotes`, reply via `replyNote`, verify the reply and answered state.
- Test phone return from sleep/offline. The page reconnects and refreshes; an expired session requires sign-in again.
- Delete only your test objects/logs. Do not reset live state indiscriminately.

## 6. Use it as a work record

Create the project and individual build steps before work starts. Mark a step in progress as it starts; update facts, tests, blockers and decisions as they happen; resolve only after the step's outcome is checked. Put the final URL and summary in Result and attach relevant screenshots. A live page is not the same as a fully verified page: publication and verification can be separate steps.

Use concise decision summaries written for the owner. Do not publish private internal traces, credentials or unrelated material. Do not invent timestamps, progress or test results. Label elapsed time accurately: this template's time-since-start includes pauses and is not an active-work timer.

## Deployment pitfalls

- GitHub Pages builds are asynchronous. Wait for success, then inspect the actual served HTML/assets and rendered page. A commit alone is not delivery.
- Version frontend assets when changing their contract. Existing open tabs need one reload for a new frontend release; data changes then arrive live.
- Durable Objects may keep old code for up to five minutes under the default deferred update strategy. This template asks for immediate updates. See the current provider docs if your Wrangler version does not support that field.
- KV is eventually consistent. Do not use KV as a low-latency version bus. This template uses strongly consistent Durable Object storage instead.
- Use hibernating WebSockets, not an ordinary connection that keeps the object active indefinitely. Broadcast only a version; authenticated clients fetch state.
- Handle sign-in failures visibly. Do not hide the entire page until sign-in succeeds; keep error text available even when external assets fail.
- Never copy account cookies, passwords or OAuth tokens into public examples. Use supported provider sign-in flows.

## Free-tier sizing

The default 30-second visible-tab safety check is 2,880 requests/day for a tab left visible 24 hours. Add real updates, reconnects and other tabs. Workers and Durable Objects currently include 100,000 requests/day each. Hibernation avoids idle duration usage; outbound WebSocket messages do not incur request charges. KV sign-in sessions consume reads on actual authenticated operations, not every safety version check.

Preview gallery: up to 60 images / 3 MB total in this starter, each <=110,000 characters. Requests/replies have the same per-image limit. Keep notes/images modest and archive old material. These are starter-level guardrails, not a complete quota management system. Free-tier excess can fail requests; no paid upgrade is enabled by this code.

Sources to recheck:
- https://developers.cloudflare.com/workers/platform/pricing/
- https://developers.cloudflare.com/durable-objects/platform/pricing/
- https://developers.cloudflare.com/durable-objects/deployments/durable-objects-code-updates/
- https://developers.cloudflare.com/kv/platform/limits/
- https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid
- https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
