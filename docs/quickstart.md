# Quickstart

This community bridge uses a pinned Harness runtime and MeetStream's hosted MIA voice tools. For a guided first run, use the [plain-language quickstart](../quickstart.md).

## Prerequisites

- Node.js 24.9 or newer and Git.
- DeepSeek Harness **0.2.1-rc.5**. Install `npm install -g @deepseek-ai/dsh@0.2.1-rc.5`, or use that version of the official macOS desktop app. The launcher detects its bundled CLI automatically.
- A DeepSeek API key with model access and API usage enabled, or a signed-in Harness desktop account configured for programmatic inference.
- MeetStream API key with MIA configuration access; the selected realtime voice provider connected in the MeetStream dashboard. The default is Google Gemini; its billing is separate from DeepSeek inference. Also connect Deepgram for the bot's speaker-labelled live transcription (`deepgram_streaming`, `nova-2`). The bridge requests 48-hour media retention.
- HTTPS ingress that forwards HTTP requests to `127.0.0.1:8787`. By default the launcher manages an installed ngrok CLI using `NGROK_AUTHTOKEN`; leave `PUBLIC_URL` blank for that path. To manage ingress yourself, run `ngrok http 8787` separately and set its HTTPS origin in `PUBLIC_URL`. Keep `/admin/*` excluded at your reverse proxy. All admin routes also require a separate random operator token that is never sent to MIA.
- A meeting host able to admit the bot, allow recording/transcription and enable chat. Zoom can require Marketplace setup and recording permission. See [MeetStream platform FAQ](https://docs.meetstream.ai/guides/help/faq).
- A disposable clone of the repository with no secrets and no unrelated changes. Agent tools run under the current OS user with Harness sandbox enforcement. Do not point this preview at a sensitive working checkout.

## Configure

```sh
npm ci
cp .env.example .env
chmod 600 .env
```

Edit `.env` privately:

```dotenv
REPO_PATH=/absolute/path/to/disposable-clone
MEETING_URL=https://meet.google.com/xxx-xxxx-xxx
PUBLIC_URL=https://your-tunnel.example
MEETSTREAM_API_KEY=your-workspace-key
# Option A: use your own API key
DEEPSEEK_API_KEY=your-deepseek-key
# Option B: reuse the installed Mac Harness account instead
# DSH_ACCOUNT=true
```

For API-key authentication, leave `DSH_ACCOUNT=false` and `DSH_LOCAL_OLLAMA=false`. The bridge explicitly selects the native `deepseek-official` route and `deepseek-flash` model. `DEEPSEEK_API_KEY` must have API usage and model access enabled. Missing keys and conflicting modes fail before any bot/configuration is created.

The account option mounts the official credential and account plugins through `config/account.yml`; it never reads Chrome cookies. It currently identifies the installation as macOS. The API-key path is the portable option. Do not configure both account mode and an API key expecting fallback: provider errors never silently switch providers.

```sh
npm run doctor
npm run doctor -- --probe
```

The probe sends a small real inference request and can incur usage. Resolve missing credentials, expired login or insufficient balance before joining a meeting. Every normal launch checks the exact CLI version and makes one small read-only inference check, bounded to 60 seconds, before inviting the bot. This uses your selected provider and can incur usage. Missing or unusable model access stops startup before MIA or bot creation.

## Join and talk

```sh
npm start
```

Missing credentials are prompted with hidden input and saved privately in `.env`, along with the repository path. The meeting link is prompted per run unless configured explicitly, or supplied with `npm start -- --meeting URL`. If `PUBLIC_URL` is blank, the launcher asks for an ngrok authtoken and manages its own connection. `npm start` installs missing bridge dependencies; Harness and ngrok remain one-time prerequisites. It starts the local service, creates an owned MIA configuration, then creates one meeting bot. Admit **DeepSeek Meeting Agent**.

1. Say “DeepSeek, ask the repository where the parser is implemented.” Fast answers return to MIA within a 20-second bridge wait and can be spoken directly. Longer questions return a task reference, then post the result to chat; ask “DeepSeek, what is the result for that task?” for the spoken answer.
2. Discuss the reproducible bug. Final transcript events become bounded, speaker-labelled context. Transcript events alone never launch tasks.
3. Say “DeepSeek, propose investigating the empty-input parser crash.” This creates an `awaiting_approval` task and executes nothing.
4. In a second terminal, inspect the exact request and approve it:

```sh
npm start -- status
npm start -- approve
# Or approve a previously reviewed exact reference:
npm start -- approve SESSION_ID TASK_ID
```

The operator command approves exactly that task in that configured repository. MIA cannot call approval: its credential cannot access admin routes. A spoken “yes” alone is deliberately insufficient in this preview, because MIA-generated arguments cannot prove participant authorization.

Engineering tasks run asynchronously in workspace-write mode. Short status and completion messages go to meeting chat through MeetStream's `send_message` REST endpoint. MIA owns its speech WebSocket; the launcher must not override it. Results remain in the private ledger even if chat delivery fails. Ask the agent for `task_status` to hear them. Large results are explicitly marked `resultTruncated` when shortened to fit the voice-tool response cap; the full retained result remains available through local status. Voice is delivered by MIA's separate realtime model; DeepSeek performs repository reasoning and engineering execution.

To cancel, say “DeepSeek, cancel task [reference].” Cancellation aborts the owned process group, escalates to SIGKILL after two seconds if needed, waits for process closure, and suppresses stale results. It does not undo already-written files. Failed or cancelled Harness sessions are discarded before a later task, so their unfinished instructions are not resumed. A small IPC supervisor also terminates the runtime when its bridge parent disappears.

## Stop and recover

Press Ctrl+C in the launcher or run:

```sh
npm stop
```

Shutdown also stops a startup prompt or inference check. It closes only its own managed tunnel, and cancels pending and active tasks, asks the owned bot to leave, polls briefly for terminal status, then deletes its owned MIA configuration. A successful stop also waits for the service to release its state so it can be restarted immediately. If confirmation fails, cleanup stays pending in `.state/bridge.sqlite`; retry `npm stop` or the next launch. Never delete `.state` to work around a pending bot.

SQLite holds the single-owner lock, which the OS releases automatically after a crash. A new launch first reconciles previous sessions. Interrupted or queued work from a crashed process is marked interrupted and never replayed. Proposals remain historical and cannot run in a closed session. Create a new explicit request after reviewing the checkout.

If a create call timed out before its returned ID was saved, ownership remains ambiguous. The launcher refuses to start another bot. Locate the configuration named `DeepSeek bridge SESSION_ID` and bots with `custom_attributes.deepseek_bridge_session=SESSION_ID` in MeetStream; remove only these resources and reconcile the private ledger after confirming removal. This preview intentionally requires operator recovery for this rare case.

## GitHub handoff

The configured clone supplies repository context. The bridge returns findings and any draft PR URL produced by a separately authorized workflow; it does not automatically push code or open PRs. To publish an engineering result, inspect its diff and tests first, then use the explicit GitHub handoff documented in [GitHub workflow](github.md). This keeps publishing credentials out of agent subprocesses.

## Offline checks

```sh
npm test
npm run check
npm run demo
npm audit --audit-level=high
```

The demo uses a deterministic fixture agent and fake meeting chat. It is not evidence of live DeepSeek inference, audible output or meeting-platform compatibility.

## Provider and sandbox contract checks

```sh
npm run test:api-key
npm run test:sandbox
```

The API contract check uses deterministic DeepSeek Messages responses and a simulated meeting API while running the actual pinned Harness. It covers model routing, session resume, exact task approval, real tool edits, independent regression tests, result delivery, duplicate receipts, authentication/usage errors and the npm lifecycle. The sandbox check exercises actual Bash and file-write tools in both permission modes.

Use `npm run doctor -- --probe` to check your own selected provider independently. Every normal start also performs its bounded provider check before inviting a bot.
