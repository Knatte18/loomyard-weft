# Refresh the README: outdated parts, the dead Autoboard link and a shorter Lineage

`readme-refresh` · Note · enhancement, docs, slack

## Seen (operator, 2026-11-11)

The operator read the README and found it already somewhat out of date.
Slack is not mentioned in it at all, though `lym` has a Slack relay.
The Lineage list links `https://github.com/willietran/autoboard` (`README.md`, Lineage section), and that repository is gone: the GitHub API answers 404.
Autoboard was the largest inspiration for Millhouse, so the credit should stay.
LoomYard has also grown far beyond Millhouse, and the Lineage section no longer shows that.

## Idea

Refresh the README as one pass.
Read it against the current `lym` verbs, `docs/overview.md` and `docs/getting-started.md`, and fix what no longer matches.

Add Slack, with the reason a reader will ask for: why a coding-agent tool needs Slack.
The relay lets an operator follow the orch and answer it from a phone, through Slack, which the organisation already uses and has approved.
The section states that need and what the relay does; it does not describe the setup, which belongs in `docs/`.

In Lineage, keep Autoboard and Willie Tran as plain text without the dead link, with a note that the repository was removed.
Shorten Lineage to the origin and the credits, now that LoomYard stands on its own.
