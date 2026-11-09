# MeetStream × DeepSeek Harness

A community bridge that brings a DeepSeek Harness agent into a meeting through MeetStream. Ask questions about a configured Git repository, propose engineering work, approve the exact task, and receive answers through voice and meeting chat.

This project is independent of DeepSeek and MeetStream. It pins Harness **0.2.1-rc.5** and keeps engineering execution behind a local operator approval step.

## Start

Follow the [quickstart for non-technical users](quickstart.md) for the one-time prerequisites and account setup. Then run:

```sh
npm start
```

The launcher installs bridge dependencies, asks privately for missing settings, checks your configured model access, opens its own ngrok connection and invites **DeepSeek Meeting Agent**. Admit the bot in the meeting and keep the terminal open.

From another terminal in this folder:

```sh
npm start -- status
npm start -- approve
```

End with **Ctrl+C** in the original terminal or `npm stop`. Shutdown cancels active and unapproved work, removes the owned bot and MIA configuration, and closes the bridge's own connection. It also works during startup.

## Behavior

- Hosted MeetStream MIA routes explicit requests and handles voice. DeepSeek Harness performs repository reasoning and tool execution.
- Questions use a read-only Harness session. Engineering proposals require operator approval and use a separate workspace-write session.
- Speaker-labelled final transcripts supply bounded context; a transcript callback alone never starts a task.
- SQLite stores meeting/session mapping, stable request receipts, task references, results and chat-delivery records. Identical request IDs reuse the same task; conflicting reuse is rejected.
- One execution slot protects each meeting and repository. Cancellation waits for the runtime to stop; crashed work is never replayed automatically.
- Start, progress and completed results are sent through MeetStream's chat API. Ask for a task's status to hear an asynchronous result aloud.
- Cleanup touches only owned resources and retains unresolved state for retry. Publishing code and opening PRs remain a separate reviewed handoff.

MeetStream supports Google Meet, Zoom and Microsoft Teams. Each platform has its own admission and account prerequisites; see [compatibility and checks](docs/verification.md).

## Documentation

- [Quickstart](quickstart.md)
- [Technical configuration](docs/quickstart.md)
- [Architecture and trust boundaries](docs/architecture.md)
- [Compatibility and checks](docs/verification.md)
- [Meeting walkthrough](docs/demo.md)
- [GitHub review and PR handoff](docs/github.md)
- [Developer article](docs/blog.md)
- [References](docs/references.md)

## Development

```sh
npm ci
npm run check
npm run format:check
npm test
npm run test:api-key
npm run test:sandbox
npm audit --audit-level=high
```

The automated suite exercises bridge logic and transports. The API-key contract check drives the actual pinned Harness against deterministic API fixtures, including tool execution, approval, retries, authentication errors and shutdown. `npm run doctor -- --probe` checks the provider configured on your machine.

MIT licensed. See [LICENSE](LICENSE).
