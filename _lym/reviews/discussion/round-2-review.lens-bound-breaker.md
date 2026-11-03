```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guarantee and claim about existing code in the decision record.

**Claims attacked**

- D6 trust-edge bound, "lym trusts only worktrees it launches in… Prepare's Warn names a committed `.codex/config.toml`": **broken**. A trusted project's `.codex/rules/*.rules` also load, and an allow rule there lifts the sandbox.
- Constraint "lym renders no rules file into its home" as the sandbox's only exemption source: **broken**, same counterexample.
- D2 "lym creates [a /tmp directory] with mode 0700 named for a hash": **broken**. The name is predictable in a shared `/tmp`, and no owner or real-directory check is stated.
- D2 "Codex writes under /tmp, a writable root": held on Linux (spike line 283). Unproven on macOS, which the record itself lists as an Open risk.
- D1 bound "the chair's wait… always ends": held at the defaults (assist 60 min against chair 240 min). **Broken** when `assist_reviewer_timeout_min` plus start retries reaches `review_timeout_min`. The chair then times out inside its wait, and nothing bounds one key against the other.
- D1/D2 "a retry never shares the first attempt's files" (token `<round>b`): held. `burlerreview.go:254-256` mints `2b`, and D1 removes stale files before the chair starts.
- D1 "probe/resume derives the same files from the record": held as a design statement. Today's `probeLiveReview` (`burlerreview.go:415`) always uses `strconv.Itoa(round)`, and the record's change to read the recorded token covers that.
- D2 "a reboot that clears /tmp loses no done report": held. The kept copy under `.lym` is written before the done marker.
- D13 "Codex 0.164.0 writes auth.json in place": held. `storage.rs:234-235` uses `truncate(true).write(true).create(true)` with no rename.
- D13/D5 "a logout removes the link and Prepare creates it again": **partly broken**. After a logout, a login inside a lym-home Codex session writes a regular `auth.json`. Prepare only "creates any missing link", and `fslink.prepareLink` refuses to clobber. The lym home then holds an auth that diverges from the operator's, silently.
- D5 "two concurrent Prepares lose no root" (`internal/lock`), and the bound that a Codex write racing the render is overwritten: held, as stated.
- D6 "a live Codex whose first turn never starts… is not retried": held. Shuttle retries only a start whose teardown ended died (`shuttleengine/doc.go:533`), and a never-ready start ends at the startup timeout. The support log contradicts this at line 61 ("which shuttle retries"), but the support log has no downstream reader.
- D6 resume line "starts no Codex": held. A gated assistant carries no done-when list, so reed relaunches the resume line, which only exits the shell.
- D7 hook fail-open bound: held. D9 tail-miss bound: held. D10 "a Codex subagent's commands meet no hook": held (Constraints, spike).

**Findings**

1. MEDIUM · design · D6 Bound and Constraints ("lym renders no rules file into its home"). The trust-edge bound understates what automatic trust lets through. A trusted project layer's `.codex/rules/` is loaded into the exec policy (Codex `rust-v0.164.0` `codex-rs/core/src/exec_policy.rs:674-686`, `load_exec_policy` iterates User and Project layers). The spike shows an allow rule runs its command outside `workspace-write` (`docs/research/codex-spike-2026-10-24/report.md:288-290`). So a committed `.codex/rules/x.rules` with `prefix_rule(pattern=["bash"], decision="allow")` lets the assistant write anywhere on disk with network access. Prepare's Warn names only `.codex/config.toml`, and the interactive TUI has no `--ignore-rules` (spike line 291). D6's bound must state this and widen the Warn, or refuse a worktree holding `.codex/rules/`.
2. MEDIUM · design · D2 report directory under `/tmp`. Counterexample: another local user, or an earlier lym version, pre-creates `/tmp/<hash>` or plants a symlink there. The name is derivable from the review path and token. lym's create then either fails or, with MkdirAll-like semantics, accepts a directory it does not own. The assistant would write the report into it, and the gate would read from it. D2 states no ownership or real-directory check and no removal after the round, so a directory leaks per attempt with no sweep. Contrast `PATTERN-lym-temp-root`'s root rule ("refused unless it is a real directory owned by the user", `pattern/PATTERN-lym-temp-root.md:6`).
3. LOW · design · D1 Bound / D3 `assist_reviewer_timeout_min`. The "one wait that… always ends" claim depends on the assistant's start retries plus its timeout plus one await cap staying under the chair's `review_timeout_min`. The defaults are 60 against 240 (`internal/loomengine/template.yaml:24`), but nothing validates the relation. With an assist timeout of 240 the chair times out mid-await, and the round dies or retries for a non-gating seat.
4. LOW · design · D13/D5 "Prepare creates it again". After a logout removes the link, a login in a lym-home Codex session creates a regular `auth.json`. Prepare treats the link as present and does not replace it: it creates only missing links, and `fslink` `prepareLink` refuses to clobber (`internal/fslink/fslink.go`, `prepareLink`). The lym home then refreshes its own token, apart from the operator's, with no Warn. Prepare should check that `auth.json` is a link to the operator's file and Warn or refuse otherwise.
5. NIT · consistency · support log line 61. It says a turn that never starts "fails the start, which shuttle retries". D6 says it is "not retried", which matches `internal/shuttleengine/doc.go:533` (only a start that ended died is retried).

**Tool calls**

- Grepped `review_timeout_min` in the template YAMLs: the context lacked the chair's timeout default needed to test D1's wait bound.
- Grepped `shuttleengine/doc.go` for startup and retry, then for `ErrNotStarted`: the context lacked shuttle's start-retry rule needed to test D6's "not retried" claim.
- Grepped `shuttleengine/doc.go` for `retr`: the same rule; this call found line 533.
