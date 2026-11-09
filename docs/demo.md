# Meeting walkthrough

Prepare a disposable Git checkout with a small reproducible bug and its regression test. Configure the provider accounts described in [quickstart](../quickstart.md).

1. Join a meeting, run `npm start`, and admit **DeepSeek Meeting Agent**.
2. Discuss the bug and confirm that final transcript entries have speaker labels.
3. Ask “DeepSeek, where is this parser implemented?” Confirm the read-only answer in chat and voice.
4. Ask “DeepSeek, propose fixing the parser crash and running its tests.” Confirm the proposal is awaiting approval and the files remain unchanged.
5. In a second terminal, run `npm start -- approve`, read the exact request and select its number.
6. Confirm start/progress/result messages in chat. Independently inspect the changed files and run the repository's test.
7. Ask for the task's result aloud. Use its saved reference for follow-ups.
8. Propose and approve a controlled long-running task, then ask the assistant to cancel it. Confirm cancelled status and actual process termination.
9. Leave one proposal unapproved and run `npm stop`. Confirm that it never executes, the bot leaves and cleanup completes.

If checking retries programmatically, preserve the request ID and verify that an identical delivery returns the existing task instead of executing twice.

Record the meeting platform, runtime version, voice provider, task references and observed outcomes. Redact credentials, meeting URLs and participant information before sharing a recording. A recording should accurately show the workflow and environment used.

For a deterministic developer example without a meeting, run `npm run demo`. Its agent and meeting chat are fixtures.
