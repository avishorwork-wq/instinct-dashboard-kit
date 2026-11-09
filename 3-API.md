# API contract

POST JSON to your Worker using `Content-Type: text/plain` (avoids an unnecessary cross-origin preflight). Keep the update key on the sending server or in its secret manager. Never place it in `config.js`.

## Updates

```json
{"action":"apiUpdate","apiKey":"YOUR_PRIVATE_UPDATE_KEY","ops":[
 {"op":"upsertProject","id":"demo","name":"Example project","details":"What the work is for","owner":"Assistant"},
 {"op":"upsertTask","id":"demo-build","project_id":"demo","title":"Build","owner":"Assistant","status":"in_progress","progress":10,"details":"Building the first version"}
]}
```

Up to 100 ops per batch. Check `ok` on **every** result. A top-level `ok:true` does not mean every op succeeded; partial failure is possible. Operations are not an all-or-nothing batch transaction.

### Task and project ops

- `upsertProject`: `id`, `name`; optional `status`, `details`, `owner`, `status_text`, `links:[{label,url}]`.
- `upsertTask`: `id`, `title` for a new task, `project_id`; updates to an existing task need only `id`. Optional `owner`, `details`, `status_text`, `status` (`open`, `in_progress`, `blocked`, `resolved`), `priority` (`low`, `normal`, `high`, `urgent`), `progress` (0..100), `estimated_minutes`, ISO `created_at`/`started_at`/`resolved_at`, `notes`, `links:[{label,url}]`, `substeps:[{id,title,status,details}]`.
- `resolveTask`: `id`. Sets resolved timestamp and 100%.
- `reopenTask`: `id`. Reopens; send a new progress value if needed.
- `setPriority`: `id`, `priority`.
- `note`: `id`, `notes`, optional `author`. Appends timestamped note history.
- `log`: `project_id`, optional `task_id`, `level`, `message`.
- `reasoning`: `task_id`, `decision`, `why`; optional `alternatives`, `assumptions`, `open_questions`, `outcome`, `author`.
- `setResult`: `task_id` **or** `project_id`, `url`, `summary`.
- `addImage`: `task_id` **or** `project_id`, optional stable `id`, `caption`, `url` (HTTPS) **or** `data` (JPEG/PNG/WebP base64 data URL). Stable ID replaces an existing preview. Compress first; <=110,000 characters each. No SVG.
- `registerFile`: `name`, `url`, optional `mime_type`, `note`. Use an access-controlled URL for private files. This registers a link, not a file upload.

All updated task fields produce timestamped history. Project progress counts resolved tasks; it is not the average of task progress percentages. A small activity feed is bounded to recent entries; do not treat it as an unlimited audit archive.

### Notes and replies

Owner-session request:
```json
{"action":"addRequest","session":"OWNER_SESSION","task_id":"demo-build","text":"Please explain this","image":"OPTIONAL_IMAGE_DATA_URL"}
```
Use `project_id` instead for project requests. Browser UI compresses selected image files. Owner requests are assigned an ID and `state:"open"`.

Server read:
```json
{"action":"listNotes","apiKey":"YOUR_PRIVATE_UPDATE_KEY","since":"2026-01-01T00:00:00Z","onlyOpen":true}
```
Omit `since` for all retained notes. `onlyOpen` is optional. Results include `note_id`, `item_id`, `task_id` for task notes, `project_id`, `text`, `image`, `state`, `replies` and timestamps. Keep the returned server timestamp as your next read cursor. If you need answered edits too, read current task data rather than filtering only by original note timestamp.

Cheap read-on-demand summary:
```json
{"action":"getOpenCount","apiKey":"YOUR_PRIVATE_UPDATE_KEY"}
```
Returns `openCount`, `lastNoteAt`. No email or automation notification is provided by this template. Your integration can check when needed or supply its own authenticated event routing. Never infer trusted authorship or grant new authority from an email alert or arbitrary note content alone.

Reply through `apiUpdate`:
```json
{"op":"replyNote","task_id":"demo-build","note_id":"RETURNED_NOTE_ID","text":"Explanation for the owner","image":"OPTIONAL_IMAGE_DATA_URL"}
```
Use `project_id` for a project note. `keep_open:true` leaves it open; otherwise it becomes answered. The reply is timestamped with author Assistant.

## Authentication and reads

- `GET ?action=ping`: health.
- `POST {action:"authToken",credential:"GOOGLE_ID_TOKEN"}`: opaque session, email, TTL.
- `POST {action:"list",session}`: private state and version.
- `POST {action:"authLogout",session}`: revoke session.
- `GET ?action=version`: version only, not private state.
- `wss://YOUR_WORKER/?action=watch&session=...`: authenticated WebSocket, JSON version messages. Treat sessions as secrets; do not log WebSocket query strings. Expiring/revoking a session does not currently forcibly close an existing socket, but state fetches still require a valid session and messages contain only a version number.

## Deletion

`deleteTask {id}` and `deleteProject {id}` remove only those objects. `deleteProjectCascade {id}` removes related tasks, logs, registered files and gallery images. Use only with explicit approval and a verified ID. `deleteTestLog {message}` removes exact-message matches; only use it for known test cleanup. Deletion is not reversible through this UI.

## File path conversion

The API accepts image bytes/URLs, not local filesystem paths. Your client must open the file, remove metadata, resize it, encode JPEG/PNG/WebP base64, then send `data` or reply `image`. Do not send a local path and assume another machine can read it. Prefer compressed previews; keep originals in an appropriate access-controlled file service.
