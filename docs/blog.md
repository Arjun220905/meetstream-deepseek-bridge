# DeepSeek meeting agent: connect Harness to MeetStream

A DeepSeek meeting agent can connect an engineering discussion to repository investigation: MeetStream provides meeting presence and voice, and DeepSeek Harness runs the repository task. Harness is an [open-source agent runtime](https://github.com/deepseek-ai/deepseek-harness). This community bridge keeps tasks explicit, records their references, and requires local operator approval before engineering work executes. The integration pins its runtime version and keeps a human operator in control of engineering work.

## Contents

- [What does the bridge connect?](#what-does-the-bridge-connect)
- [How does a conversation become a task?](#how-does-a-conversation-become-a-task)
- [How do you launch it?](#how-do-you-launch-it)
- [What happens when a task finishes or fails?](#what-happens-when-a-task-finishes-or-fails)
- [How do you check your setup?](#how-do-you-check-your-setup)
- [FAQ](#faq)

## What does the bridge connect?

[MeetStream's meeting bot API](https://docs.meetstream.ai/api-reference/introduction) supplies the call connection. Hosted MIA routes explicit participant requests to authenticated custom functions. Harness receives the repository request and relevant discussion, runs its tools, and returns a result to the bridge.

The [MIA API](https://docs.meetstream.ai/guides/mia/mia-api-guide) lets a configuration register HTTPS custom functions. This project registers four: ask a repository question, propose engineering work, retrieve task status and cancel a task. The local operator has a separate approval route that is never exposed as a MIA tool.

Voice and repository reasoning use separate providers. The voice model routes requests and speaks tool responses. Harness executes repository work using its configured model. Use programmatic API credentials or the explicitly configured Harness account provider.

## How does a conversation become a task?

Imagine Alice reproduces an empty-input parser crash while Bob explains which component calls it. Their final, speaker-labelled transcript entries become context. Hearing the discussion does not execute anything.

Alice then says, “DeepSeek, propose investigating that parser crash.” MIA calls `propose_task`; the bridge records an `awaiting_approval` task. The operator reviews its exact text and approves its ID:

```sh
npm start -- status
npm start -- approve SESSION_ID TASK_ID
```

The bridge runs that task in workspace-write mode. Repository questions use a separate read-only Harness session. A spoken “yes” cannot approve a proposal in this preview: model-generated arguments cannot prove who authorized the work.

### Why keep a request receipt?

Network retries are normal during calls. The bridge stores a receipt before dispatch and binds it to the session, request ID and arguments. An exact retry gets the existing task; changing the arguments under the same ID produces a conflict.

This prevents duplicate execution when retries preserve their IDs. It does not recognize two different IDs as the same spoken request. Engineering work still requires operator approval of each proposal.

## How do you launch it?

Install Node.js 24.9 or newer, Git, Harness `0.2.1-rc.5` and ngrok once. Have usable model access, a MeetStream key and connected voice/transcription integrations ready. The launcher manages HTTPS ingress with ngrok unless a public URL is configured explicitly. The [plain-language quickstart](../quickstart.md) walks through this setup.

```sh
npm start
# Follow the private setup prompts. Bridge dependencies install on first use.
# Startup checks model access before inviting the bot.
# Later, in a second terminal:
npm start -- approve
# End with Ctrl+C in the first terminal, or:
npm stop
```

Use a disposable repository clone without secrets. Provide a real Google Meet, Zoom or Teams link and admit the bot if host approval is required. Consult the [platform prerequisites](https://docs.meetstream.ai/guides/help/faq), particularly Zoom setup and recording permission.

The supported Harness version is pinned because this upstream project is evolving. The super-agent prototype informed the repository workflow, but the bridge does not mount it or rely on its autonomous polling behavior.

## What happens when a task finishes or fails?

The bridge stores the result before attempting meeting chat delivery. Start and completion notifications use MeetStream's `send_message` REST endpoint. MIA retains its own speech connection; supplying an external control socket alongside its configuration can leave an agent silent. Fast questions can return their result within a 20-second tool wait for MIA to speak; a participant can also ask for a longer task's result to have MIA read it aloud. Automatic background voice announcements are not implemented.

Cancellation marks the task cancelled, terminates the owned runtime process and suppresses late results. It does not undo file edits already made. On a crash, uncertain running or queued work becomes interrupted and is never replayed automatically.

Ending the session cancels pending work and requests bot departure. The bridge confirms terminal bot status before deleting its owned MIA configuration. If confirmation fails, the cleanup record remains for a later retry.

## How do you check your setup?

Startup checks the configured model before inviting the bot. `npm run doctor -- --probe` performs the provider check independently. The automated developer suite checks receipts, exact approval, queues, process cancellation, protocol failures, transport authentication and cleanup ordering with controlled API responses.

Use the [meeting walkthrough](demo.md) to check speech, transcript attribution, asynchronous chat, cancellation and departure with your meeting host and provider configuration. Publishing a draft PR remains a reviewed operator handoff.

## FAQ

### Is this an official DeepSeek product?

No. It is a community bridge using the open-source DeepSeek Harness runtime and MeetStream APIs.

### Can I use free DeepSeek browser chat as the task backend?

This bridge does not automate browser chat. Its backend is the Harness CLI, which needs a working model provider. Configure the provider through its API or the supported Harness account mode.

### Does every sentence start an agent task?

No. Transcript callbacks only update context. Explicit custom-function requests create questions or proposals, and engineering proposals require local approval.

### Will restarting the bridge repeat a partially completed task?

No automatic replay occurs. Review the interrupted task and its repository changes before creating a new request.

Explore the [MeetStream developer resources](https://meetstream.ai/developers) to build and test your own meeting-to-execution workflow.
