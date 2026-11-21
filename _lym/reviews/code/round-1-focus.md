---
round: 1
focus:
  - "internal/pullrequest/client.go `rulesFeatureUnavailable`: does the 403 plus message-prefix match narrowly enough that a real token failure still errors, and does it survive the error wrapping and the nil-response case?"
  - "internal/pullrequest/client.go `RequiresPullRequest` and `RequiredChecks`: is the plan-gated early return correct on a later page of a paginated read, where earlier pages already gathered rules or contexts?"
  - "Scope against the Goal: the entry asks that the plan-gated 403 read as no rules, or that the halt name the real cause. Does anything else in the diff (landingshed doc, sandbox-3 report edit) go beyond that, and does any caller still map this 403 to a token failure?"
  - "internal/pullrequest/client_test.go `TestClient_Calls`: do the `plan` and `forbidden` cases check the behaviour their names state, and was any existing assertion for `unknown` loosened?"
  - "Doc comments in internal/pullrequest/doc.go and internal/landingshed/doc.go: do they state what the code now does, and does the repository's constraints document require any other doc to change with this behaviour?"
  - "Commit messages of the range: do any carry an agreement or improvement claim without before and after figures, or a Root-Cause trailer naming a symptom?"
---

Seed focus from the decision record's Goal and the committed diff over `46a316e0`.
