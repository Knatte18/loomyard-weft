```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the decision that asks for it, and each decision to the changes that build it.

**Decisions** (the decision record alone is the answer key):
- **G1:** find why a live beam orch reads as down: the liveness read, the strand's record in reed's state, or both. The Added-after-Discussion entry narrows this to "no evidence can be read; name the cause instead".
- **G2:** fix the read so a live orch reads as live. The Added-after-Discussion entry overrides this: "Do not build (a)", so no change to the liveness rule.
- **G3:** an orch whose strand is missing from reed's state reads as down with that cause named.
- **AaD** (parent, 2026-11-28): build (b). Every down reading of an orch names its cause, in `lym orch status` and in the routing's log line. The causes are pane dead or unlisted, pane in another session, start shell past the grace, and strand missing from reed's state.

**Changes and what they trace to:**
- **C1, reed down cause.** `DownCause` with its constants and `downCause` in `internal/reedengine/apply.go:79-111`, `StrandStatus.Down` in `lifecycle.go:87-88`, its use in `Status` at `lifecycle.go:1333,1347`, the `doc.go:87` line, and the `lifecycle_test.go` `wantDown` assertions. Traces to AaD: the reed causes pane-dead, pane-unlisted, pane-elsewhere and start-shell.
- **C2, orchroute reading and log line.** `Reading`, `ReadingOf`, `readingFrom`, the three cause constants and the Info/Debug log in `Live` (`internal/orchroute/live.go:18-100`), the `doc.go:3` line, and the `live_test.go` rows. Traces to AaD (the routing's log line, every cause) and to G3 (`DownStrandNotInReedState`).
- **C3, the `strand_down` field.** The `strand_down` envelope field and its help text in `internal/orchcli/status.go:172-174,220`, and the `cli_test.go` rows. Traces to AaD (named in orch status) and to G3: the new row "a strand missing from reed's state reads down with that cause".
- **C4, `OrchLive` doc line.** The new doc line at `internal/orchcli/live.go:65`. C2 forces it, since `OrchLive` delegates to `Live`, which now logs.
- **C5, rewritten "a pane in another session" fixture.** The row in `cli_test.go:328-333` now has `Live` false and `Down` pane-elsewhere. C1 forces it: reed never reports a pane in another session as live, so the old fixture's `Live: true` no longer matches what reed produces.

**Decisions and the changes that build them:**
- **G1:** C1–C3, as the amendment directs: no root cause is claimed, and the cause is named instead.
- **G2:** no change, and that is correct. The amendment withdraws it, and the diff leaves the liveness rule as it was (`downCause` derives from `aliveIDs` without changing it).
- **G3:** C2 and C3.
- **AaD:** C1, C2 and C3. Every reed cause the entry lists has a constant. `no-strand-recorded` and `no-reed-session` go beyond the entry's list, but "every down reading names its cause" needs them, so they trace to AaD.

**Findings:** none from this lens. Every change traces to a decision or is forced by one, and every decision still in force has changes that build it.

Two points that border scope belong to the handler or other lenses:
- `no-reed-session` never shows in the `strand_down` field of `lym orch status`. The verb returns an error envelope before it reaches `ReadingOf` (`status.go:187-191`).
- The `no-strand-recorded` case logs at Debug, not Info (`live.go:66`). Whether a Debug line meets "named in the routing's log line" is a judgment call for the handler.

Tool calls: none. Everything I needed was already in context.
