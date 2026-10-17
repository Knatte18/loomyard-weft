# Card 6 — publish-older-open-stop

**Edit:**
- `internal/landingshed#Publish.Call`
- `internal/landingshed#TestPublish_OpenPullRequest`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/landingshed#FindPullRequest`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#publishGitHubServer`

**Intent:** D2: Publish stops on an older open pull request it cannot replace.
In `Publish.Call`, when `FindPullRequest` returns a nil `pr` and a non-nil `olderOpen`, Publish creates nothing, since GitHub refuses a second open pull request for the same head and base, and returns Stuck through `stuckOrCancelled`.
The reason is `withPRURL` over "an open pull request opened before this run started blocks a new one for the task branch" and `olderOpen.GetHTMLURL()`, followed by "; way forward: close that pull request on GitHub, then resume the run with `lyx loom start`", the resume verb the push-rejected stop already names.
The stop sits in the `pr == nil` branch before the change description is parsed, so nothing is created or edited.
It follows `PATTERN-refusal-way-forward`'s form by choice, with the `way forward:` clause `contracts/specs/refusal-spec.md` defines; the entry itself is not changed.

Bound: the stop applies only to an open pull request older than the run's start; an older closed or merged one never reaches `olderOpen`, so Publish creates a new pull request, and a pull request created after the run started is `pr` and takes the existing branches, so a resumed Publish refreshes its own.
A zero `RunStart` never yields `olderOpen`, so a run without a usable seed stamp keeps today's refresh of the newest open pull request.
`Publish.Call`'s doc comment and `internal/landingshed/doc.go`'s Publish paragraph name the stop and its way forward.

`TestPublish_OpenPullRequest` gains a `listBody` override and a `wantReason` field, the table asserting the exact reason on a Stuck row that sets it; its existing rows keep the single open pull request with no bound.
New row with `RunStart` `2026-06-01T00:00:00Z`:
an open pull request created `2026-01-01T00:00:00Z` with `html_url` `https://github.com/acme/proj/pull/4` and none after ends Stuck with the reason exactly "an open pull request opened before this run started blocks a new one for the task branch: https://github.com/acme/proj/pull/4; way forward: close that pull request on GitHub, then resume the run with `lyx loom start`", and zero creates and zero edits.
That the run's own pull request wins over an older open one is pinned by card 5's `FindPullRequest` row for a newer closed one above an older open one, and the stop sits inside the nil-`pr` branch, so no Publish row repeats it.

**ImpactSummary:** one producer gains one Stuck branch inside its no-pull-request path; PR-Gate, PR-Align, Finalize and the verbs ignore `olderOpen` and are unchanged.

**Commit:** 6: Stop Publish on an older open pull request it cannot replace
