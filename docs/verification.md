# Testing and compatibility

## Automated checks

Run from the project folder:

```sh
npm ci
npm run check
npm run format:check
npm test
npm run test:api-key
npm run test:sandbox
npm run demo
npm audit --audit-level=high
```

| Check | What it exercises |
|---|---|
| `npm test` | Authentication, bot/session binding, request receipts, exact approval, queues, cancellation, restart fencing, UTF-8 input, bounded output and cleanup ordering. Launcher checks cover private settings, first-start installation, command/path handling, managed connection failure and both stop methods. |
| `npm run test:api-key` | Actual Harness using a deterministic DeepSeek Messages endpoint: provider headers/model selection, resumed sessions, real tools and file edits, independent tests, duplicate protection, MIA HTTP tools, REST chat and the complete npm start/stop lifecycle. Authentication and usage failures must stop startup before meeting resources are created. |
| `npm run test:sandbox` | Actual Bash and file-write tools blocked in read-only mode and allowed in workspace-write mode. Scripted protocol responses make the check reproducible. |
| `npm run demo` | In-process workflow example with a fixture agent and simulated meeting chat. |
| `npm audit --audit-level=high` | The bridge's npm dependency tree; the separately installed Harness runtime has its own upstream dependencies. |

Tests of implementation boundaries use controlled fixtures. Service connectivity is checked against the user's configured account during startup; use `npm run doctor -- --probe` to run that check separately.

## Runtime and meeting prerequisites

Supported Harness executable: **0.2.1-rc.5**. The launcher rejects other versions until compatibility is qualified. This is a community integration with an evolving upstream runtime.

The documented setup targets macOS. CI exercises the deterministic checks on macOS and Linux. Windows full-process-tree cancellation is outside the supported scope of this release.

| Platform | Configuration to check |
|---|---|
| Google Meet | Host admission, enabled meeting chat, working microphone and configured MeetStream providers. |
| Zoom | MeetStream Zoom account/Marketplace setup, a supported invitation URL, host admission and recording permission. |
| Microsoft Teams | A supported invitation link, organization bot/access policy, admission and chat permissions. |

Platform support comes from MeetStream and does not replace validation of a particular host or organization setup. Use the [meeting walkthrough](demo.md) to check voice, speaker attribution, approved tasks, cancellation and departure in your environment.

## Operational behavior

- Transcript events add context; explicit MIA function calls create questions or proposals.
- The local operator approves exact engineering tasks. Spoken agreement alone is insufficient.
- Fast questions can return within the 20-second tool wait. Longer results are stored and sent to chat; request task status for a spoken result. Automatic background voice announcements are not implemented.
- Deduplication requires a stable request ID. Different IDs can create separate proposals, each requiring approval.
- Chat API acceptance is recorded separately from visible delivery. Ambiguous sends are not replayed automatically.
- Ctrl+C and `npm stop` cancel pending and active work, confirm departure before removing the owned MIA configuration, and close the owned connection. Keep private state until cleanup is confirmed.
- Cancellation stops runtime work but does not undo edits or external effects already performed.
- The launcher has one owner per state directory. Shared-ledger multi-replica operation is unsupported.
- GitHub publication is a reviewed operator handoff; agents do not receive publishing credentials.

For Deepgram transcription, save your Deepgram key in MeetStream **Integrations → Transcription**. The bridge requests `deepgram_streaming` under `recording_config.transcript.provider` and supplies a live-transcript callback. MIA's voice provider is configured separately under **Integrations → Models**. See the [provider documentation](https://docs.meetstream.ai/guides/transcription-recordings/providers/transcription-providers).
