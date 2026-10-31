```yaml
lens: walkthrough
outcome: reported
```

**Decisions walked:**
- D1 — sampler run, `code_review` false: Record-Write Done → Sampler-Session → Code-Bouncer Skipped by entry gate → Describe (no gate) → PR-Review → Finalize — purpose met.
- D2 — task naming `light`: profile sampler block seeds `sampler.code_review` true; no-profile task finds no `defaults` entry and runs on the hub config (`internal/runprofile/doc.go:9`) — purpose met.
- D3 — Finalize reroute with `code_review` false, non-zero budget: today `rerouteReviewed` reads only loom's budget (`internal/loomcli/landingdeps.go:392-409`); D3 states the change — purpose met as stated.
- D4 — agent runs `decision add --by operator` after a resume under a new strand: row re-records, caller rule matches current strand — purpose met; crosses D5's steer text (finding 4).
- D5 — first entry: `StartSession` with `DenyPublish` true installs the publish row, whose steer is the one fixed PR-Review text — the steer D5 promises is not what the agent gets — purpose not met (finding 3).
- D6 — normal: agent `sampler-done` (stamp 1) → file event wakes → stop session → clean + intact → Done — purpose met.
- D6 — halted + reopened session: ends Done with a live unstopped session — purpose not met (finding 1).
- D6 — flag while halted, `goto` away and back: goto entry counted from another row, stale flag ignored — purpose met.
- D7 — step at Sampler-Session: `childAtWaitingRow` reads running at listed row → no quiet notice; watchdog reading returns now — purpose met.
- D8 — board result: agent writes a note via `lym board` — purpose met.
- D9 — `reed-status-buttons` trial following `docs/sandbox-howto.md`: the procedure's first step is refused by the agent's own guard — purpose not met (finding 2).

**Findings:**

1. **MEDIUM · design · D6, entry bullet + Bound paragraph** — On entry the row ends Done on a current flag "without starting or resuming a session" and never stops a session that is already live. Walk: run halts at the row on a dead strand; the Bound paragraph allows `lym reed resume` to reopen the session; the agent runs `sampler-done` (accepted while halted, session recorded) and keeps working — a commit now leaves the checkout clean; `lym loom resume` re-enters, reads the current flag, finds the checkout clean, ends Done; the live session is never stopped and can change the checkout during Describe and Finalize. The same holds for a session an interrupt leaves running (D5's model `endSession` leaves the session live on a cancelled context, `internal/landingshed/prsession.go` `endSession`). This defeats D6's own "Describe and Finalize always start from a committed tree"; the entry path must stop any live recorded session before its checkout check, as the wake path does.
2. **MEDIUM · design · D9, the stencil bullet pointing at the sandbox hub** — The procedure the agent is sent to opens with `go build ./...` and `go test ./...` (`docs/sandbox-howto.md`, "1. Confirm the repo builds and tests green"), both refused in the agent's session by the raw-go shell guard (`internal/shellguard/rows.go` `rawGoCommandPattern`), and the doc requires every suite launcher to run "in a real, attached interactive terminal", which the agent's Bash tool is not. So the agent cannot carry out the trial D9 sends it to; the acceptance line "names no denied form" passes only because the stencil reaches the denied forms through the doc.
3. **LOW · design · D5, publish shell guard bullet** — D5 gives the session "steer text of its own" for `git push` and `gh`, but the publish row carries one fixed steer, `steerPublishDeny`, and `Spec.DenyPublish` is a bool (`internal/shellguard/rows.go:99-107, 249-253`; `internal/shuttleengine/spec.go:55-57`). Under the record as written the agent gets PR-Review's text ("commit your change and tell the operator"); D5 does not say whether it adds a row with its own switch or a spec-carried steer, either a shell-guard-table change `PATTERN-shell-guard-table` governs.
4. **LOW · consistency · D5 steer text vs D4** — D5's steer says lym pushes the branch "only after `sampler-done` ends the row", but walking D4: the agent's `decision add` commits the record, a run-record commit, and the publish row's own comment states the detached both-sides push such a commit fires carries the task branch to origin (`internal/shellguard/rows.go:247` comment on the publish row; `PATTERN-push-both-sides`). The branch reaches origin while the session is still running, so the steer text is false.

Tool calls:
1. grep for `rerouteReviewed`/`codeBudgetZero` and the darn Describe stencil — to walk D3 and D1; the manifest lacked where these are defined.
2. read `landingdeps.go` 380-420 and `wiring.go` 860-900 — to walk D3's perch-off reading and the darn wiring switch D6 replaces.
3. read `wireDarn` body and its seam consumers — to walk D6's seam wiring.
4. read `runprofile` doc and template — to walk D2's profile seed; no runprofile code in the manifest.
5. read `docs/sandbox-howto.md` head — to walk the D9 trial procedure the stencil points at.
