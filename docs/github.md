# GitHub review and draft PR handoff

This preview uses a local Git clone as the source of repository context. It does not load credentials from a GitHub repository URL, clone arbitrary model-supplied URLs, or allow meeting participants to change the configured repository. The launcher verifies that `REPO_PATH` belongs to a Git checkout.

Findings and GitHub links returned by a trusted task are retained in its durable result and sent to meeting chat. Automated PR creation is deferred; no GitHub write credential is inherited by the agent subprocess. A local operator can publish after reviewing the change:

```sh
# In the configured disposable checkout:
git status --short
git diff
# Run the repository's documented tests.
git switch -c fix/meeting-investigation
# Stage only reviewed files, with explicit paths:
git add src/path-to-reviewed-file
# Inspect staged content before committing:
git diff --cached
git commit -m "Fix reproduced meeting investigation bug"
git push -u origin fix/meeting-investigation
gh pr create --draft --title "Fix reproduced bug" --body-file /absolute/path/to/reviewed-pr-body.txt
```

The repository owner chooses the remote, branch, files and PR body. Paste the resulting draft PR link into the meeting chat. The current bridge does not automatically discover an externally created PR or attach it to an already completed task. For status followups, keep the task ID in the PR description and retain the findings in the bridge ledger.

Review the resulting diff and test output before publishing. This handoff does not grant the meeting agent GitHub write access.
