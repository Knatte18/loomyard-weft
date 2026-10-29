# Card 8 — pull-request-mode-resolver

**Create:**
- `internal/landingshed/pullrequestmode_test.go`
- `plan:internal/landingshed#Origin` -> `type Origin struct`
- `plan:internal/landingshed#PullRequestDecision` -> `type PullRequestDecision struct`
- `plan:internal/landingshed#ResolvePullRequest` -> `func ResolvePullRequest(slug, parentMode, entryField string, origin Origin) PullRequestDecision`
- `plan:internal/landingshed#TestResolvePullRequest` -> `func TestResolvePullRequest(t *testing.T)`

**Extends:**
- `internal/landingshed/doc.go`
- `PATTERN.md`

**Uses:**
- `internal/githubclient#ParseOwnerRepo`
- `internal/testkit/logcapture#CaptureVerbose`

**Intent:**
D2, D3 and D4: one landingshed function decides whether a landing goes through a pull request, so PR-Review (card 11) and Finalize (card 15) cannot disagree.
`Origin` holds the origin facts: `URL` (the task pair's origin remote URL, empty when there is none), `Upstream bool` (the parent branch has an upstream) and `PushSkipped bool` (the hub's push skip).
`PullRequestDecision` holds `Open bool`, `Reason string`, which names the values it read, and `Halt string`, a Stuck reason with its way forward, non-empty only when the landing must stop.
`ResolvePullRequest(slug, parentMode, entryField, origin)`, where `slug` is the landing run's slug, `parentMode` is the receiving branch's `beam.yaml` `pull_request` (`on`, `off`, `required`) and `entryField` the landing entry's `pull_request` (empty, `on`, `off`):
- the wanted mode is: parent `required` wants one whatever the field says; parent `on` wants one unless the field is `off`; parent `off` wants one only when the field is `on`;
- a field `off` under `required` logs one Warn naming both values and still wants one;
- not wanted is `Open: false` with a reason such as `pull request off: the parent's beam.yaml pull_request is off and the entry's pull_request is empty`;
- wanted, the origin facts are checked in order: an origin URL whose host is not `github.com`, an origin on `github.com` that `githubclient.ParseOwnerRepo` refuses, no upstream for the parent, and the push skip; the first missing fact is named;
- a missing fact where the want came from parent `on` alone (field empty) is `Open: false` with the missing fact in the reason, except that an origin on `github.com` that `ParseOwnerRepo` refuses is always the halt below, never a no;
- a missing fact under parent `required`, or under a field `on`, is `Halt`: `a pull request is required here (<which value demands it>) but <the missing fact>; way forward: give the hub a GitHub origin, an upstream for the parent branch and no push skip, then run lym loom resume`, and for a field `on` under a parent that is not `required` it adds `, or run lym batten cancel <slug> and start a new run with the entry's pull_request changed` with the told slug;
- otherwise `Open: true` with a reason naming parent mode, field and origin repository.
The resolver reads no file: its callers read the parent's `beam.yaml` and the entry and tell the values, so an unreadable `beam.yaml` is the caller's halt.
`TestResolvePullRequest` (in a new file `internal/landingshed/pullrequestmode_test.go`, `//go:build tier0`, serial since it captures the logger) is table-driven over the nine parent and field combinations on a GitHub origin with an upstream, asserting `Open` for each, the Warn for `off` under `required` and its absence elsewhere; then over the origin rows: a local-path origin under parent `on` and empty field is a no naming the origin; the same origin under `required` and under field `on` each halt with their ways forward; `https://github.com/onlyowner` halts under parent `on`; no upstream and the push skip each give a no under parent `on` and a halt under `required`.
The landingshed doc gains a paragraph on the resolver.
`PATTERN.md` gains, under `## Shed and loom`, this entry line, verbatim:

```
- `PATTERN-pull-request-mode` — Deciding whether a landing goes through a pull request: only landingshed's one resolver, from the receiving worktree's `beam.pull_request`, the landing entry's `pull_request` field and the origin; no profile or run setting carries it.
```

**ImpactSummary:** Adds a pure function and two types to landingshed with no caller yet, plus one PATTERN entry; nothing existing changes behavior.

**Commit:** 8: landingshed: the one pull-request mode resolver
