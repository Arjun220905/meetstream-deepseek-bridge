# Architecture and trust boundaries

```text
Meeting participants
    ↕ audio / voice (hosted MeetStream MIA + configured realtime provider)
MIA custom functions ── authenticated HTTPS ── Bridge HTTP service
MeetStream final transcripts ────────────────┘        │
Local operator ── separate admin credential ─────────┤
                                                    ├─ SQLite receipts/tasks/context
                                                    └─ dsh headless --json
                                                         ↕ persisted Harness sessions
                                                     Configured repository
Bridge delivery ledger ── MeetStream send_message REST API ── meeting chat
```

## State and execution

A session stores one meeting URL, repository path, owned bot/config IDs and two Harness session IDs, one per permission lane. Session adoption uses upstream's exact `--session-id` option; fresh sessions receive IDs from its `session` event. The bridge does not take over an existing Desktop UI conversation.

Each `(session_id, request_id)` has a SHA-256 fingerprint of its effective text and request kind. A SQLite transaction writes the receipt and task before execution. An exact retry returns its original task. Different arguments with the same ID produce a conflict. This is at-most-once dispatch for stable IDs, not semantic deduplication: a voice router that invents another ID for the same utterance can create another proposal. Engineering proposals still need separate operator approval.

Task states: `awaiting_approval → queued → running → completed|failed|cancelled`. Restart maps both queued and running states to `interrupted`; nothing executes merely because the process restarts. Approval records its timestamp and the exact immutable task. A meeting has a single execution slot. Meetings sharing the same configured repository are also serialized; distinct repositories have independent slots when embedded through the API; the launcher owns one active meeting.

Final transcripts are deduplicated by stable event ID or speaker/timestamp/text, capped at 40 entries and 24,000 text characters, and snapshotted into each task. Partial events and the bot's own transcript are ignored. The prompt marks meeting context as untrusted. Prompt text is not a security boundary: runtime sandbox policy and operator approval provide the actual control.

## Authentication and authorization

MIA calls carry a random bearer capability plus a session-scoped URL and bot ID. Callback URLs carry the same capability as a query parameter because arbitrary webhook headers are not assumed. Keep ingress logs free of full callback URLs, paths and payloads. Use TLS externally. Rotate the launch capability after ending the meeting.

Admin routes require a separate random token stored only in `.state/active.json` (mode 0600 under a 0700 directory). The MIA model never receives it. Configure ingress to reject `/admin/*` as additional protection. The service listens on loopback; a tunnel must deliberately forward it.

The approval actor is the local operator, not an inferred speaker name, free-form utterance, or model-generated boolean. This release does not support spoken approval or authenticated per-participant permissions. Transcript identities are context labels, not proof of identity.

Harness processes inherit a small environment allowlist. MeetStream and GitHub API tokens are excluded. DeepSeek authentication is either a model API key or the official account plugins reusing the installed account store. Account mode has no API-key fallback. Read-only questions and approved workspace-write tasks use different persisted sessions and supplied permission overlays. Permission escalation requests fail closed (`approval: never`); this does not mean full access.

Use a disposable clone without secrets. Sandboxing depends on the exact Harness installation and OS, and the bridge does not provide a container boundary or a guarantee against a malicious repository. Custom Harness plugins and profile configuration remain trusted installation inputs. Bash and file-write confinement passed the installed-runtime contract on macOS; Linux sandbox acceptance is still pending; Windows process-tree cancellation is unverified.

## Output and lifecycle

The pinned CLI's `final` event carries `text`; success also requires exit code zero and no failed durable turn-end status. Newer source snapshots include a `reason` field, which is checked when present. Invalid JSON, missing final output, failed exit, deadlines and aborted runs cannot become completed tasks. Diagnostics remain local; raw stderr and private reasoning are not sent to meetings.

The bridge first persists results, then records chat output. It marks a message submitted before sending: lost or ambiguous sends are not automatically replayed. This trades possible message loss for avoiding duplicate meeting chat. `submitted_unconfirmed` marks an in-flight attempt, `accepted` records REST acceptance, and `unknown` records failure or ambiguity. None establishes that participants saw a message. MIA retains its internally provisioned speech socket; external socket overrides are deliberately omitted. Fast questions wait up to 20 seconds to return their result directly to MIA for speech; longer runs retain their task reference. The status tool retrieves the durable result independently. Chat messages are capped at 8,000 characters; full stored results are capped at 60,000 characters. MIA responses are bounded valid JSON below the registered cap and expose `resultTruncated` when shortened.

Cancellation marks the task cancelled before aborting its process group. Repeated cancellation waits for the same running process to terminate. Failed/cancelled permission-lane sessions are cleared before another task. Late results are fenced. The runtime receives SIGTERM, followed by SIGKILL after two seconds if needed. An IPC supervisor kills the runtime group if the bridge parent disappears. Already-written files and external effects cannot be rolled back. Agent tasks are instructed not to publish or contact others; use the local GitHub handoff to publish after review.

Ctrl+C, the stop command, a terminal lifecycle callback or polling closes the session and cancels pending tasks. Removal acceptance is not departure confirmation: bot state must become terminal (or resource absence must be confirmed) before deleting its MIA configuration. Failed cleanup remains durable. Ambiguous create responses require operator reconciliation and block automatic recreation.

Private transcripts, repository paths, tasks and results remain in `.state`; Harness retains its own session history separately. There is no automatic retention purge. Deleting bridge state does not delete recordings retained by MeetStream or upstream Harness sessions. Configure provider/platform retention according to your use case.

The launcher and offline recovery acquire a SQLite exclusive transaction on `.state/owner.sqlite`. This native OS lock prevents concurrent owners and is released on process death; no PID-file recovery heuristic is used. API-key mode explicitly overrides the saved Harness provider/model selection. A shared overlay disables automatic session-title model calls and unrelated global skill inventories.

## Guided process lifecycle

The CLI creates its authenticated local control endpoint and acquires native state ownership before asking for setup values. This makes `npm stop` usable while a credential prompt or the startup model check is pending. `/admin/stop` returns HTTP 202 for intent; the stop command waits for local ownership release and independently reconciles the saved resources before reporting completion. It is not a bot-departure acknowledgement.

The optional managed ngrok connection reuses the IPC process supervisor and has a restricted environment. Ctrl+C, stop, startup failure or a lost managed connection closes owned work and preserves unresolved cloud cleanup for retry. An explicitly configured public ingress is externally owned. Prompted credentials and repository settings are saved privately; prompted meeting links are not persisted. The bounded startup inference check uses the configured provider's API and occurs before bot/config creation. Development tests are separate commands.
