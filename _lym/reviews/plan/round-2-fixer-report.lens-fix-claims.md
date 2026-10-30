```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fix added or rewrote against HEAD. Most hold. Two are slightly off, both LOW.

**Claims that hold:**
- **Card 8 allowlist test:** `TestToldGeometryInvariant_AllowlistOnly` fails on an unlisted import and on a listed one that no production file imports. `scankit.go:345` (`AssertImportAllowlistNoStale`) confirms both.
- **Card 8 comes first:** it is the first card whose landingshed production code imports `githubclient`. Of cards 1 to 7, only cards 4 and 7 touch landingshed: card 4 edits config, and card 7 only names landingshed as a holder of `API`. Neither adds a `githubclient` import.
- **Card 13 `checkHead`:**
  - `check`'s reason speaks of a merge of the parent branch and of resume (`verifygate.go:282`).
  - `check` skips with a Warn when no command is configured (`verifygate.go:225`).
  - The gate keeps the same wait marks and has no recorder: `newVerifyGate` sets no `recorder`.
  - `FailedTier` returns `(tier, build, ok)`, and `build` marks a failed build line (`runlog.go:231-234`).
- **Card 11 loomrecipe doc sentence:** it matches `internal/loomrecipe/doc.go:16` word for word.

**Findings:**

- **LOW** — card 11 Intent, the `README.md` sentence it rewrites (claim inherited from F3's proposal). It quotes the sentence as `PR-Review is a stub for a human pull-request review that is not built`. `README.md:70` reads ``and `PR-Review` is a stub for a human pull-request review that is not built.``, with backticks around PR-Review. The quoted sentence is therefore not the README's literal text, and a search for it as written will not find it.

- **LOW** — card 13, the `checkHead` sentence: "on a failed result `the pr-review gate failed at tier <n>…`, with `<n>` from `gateslot.FailedTier` over the log, else the highest tier `g.tiers(g.key)` lists". A failed result also covers a timeout (`result.TimedOut`) and a shell that could not start (`ExitCode < 0`); `verdict` handles both at `verifygate.go:276-284`. The existing code keeps those cases out of tier attribution on purpose: `recordFailure` at `verifygate.go:63` returns early for them, "since none of them names a failing test". For those cases `checkHead` would fall back to the highest tier and tell the agent the gate "failed at tier <n>", which is a guess. The sentence states a tier the code cannot back for timeouts and failed starts.

Further tool calls:
1. Read `gateslot.FailedTier`'s body, `dirtyReason`, and which of cards 1 to 7 mention landingshed, to check `checkHead`'s build/tier claim and card 8's "first card" claim.
2. Grepped what cards 4 and 7 say about landingshed, and the README line's exact text, to confirm the "first card" claim and the quoted sentence.
