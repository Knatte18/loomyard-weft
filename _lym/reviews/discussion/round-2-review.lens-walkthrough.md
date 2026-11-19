```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end and judged whether it meets its stated purpose.

**Walkthrough list:**
- D1, a discussed entry whose `decision-&lt;slug&gt;.md` is missing, rendered by a board write. Purpose met: the revision is taken over the stored entry and the file name follows the undiscussed copy. Render is handed only the copy (`render.go:44`), so the plan has to pass the originals in. The record states the requirement, so I did not raise a finding.
- D2, the page opened at `file://`, then the same bytes served at `/`. Purpose met: the CSP header blocks the `board-data.js` tag over `http` but not at `file://`.
- D3, a write on a new binary followed by sync and replay, with a second machine on an older binary. Purpose met: the bound covers every outcome.
- D4, a body with `[x](javascript:...)` and a design link at `file://` and over `http`. Purpose met, with one exception: a `design_prefix` holding a `/` (LOW finding 3).
- D5, `alt+shift+9` pressed in a task-pair window with no server running. Resolve passes on the anchor cwd, the start goes to the prime anchor, the token handshake runs, then the launcher opens the page. Purpose met, with one exception: a missing prime is described with the wrong failing call (LOW finding 2).
- D6, a hub regeneration with the hook installed. The `tasks` member is spliced after `settings` and runs `lym beam start root`. `root` is a valid slug (`internal/beamcli/cli.go:471,477`). Purpose met.
- D8, an edit from a page holding R1 while the CLI changes the entry's status. The check under the lock refuses the stale write and returns the fresh entry. Purpose met for a direct write; it fails when a live refresh happens first (BLOCKING finding 1).
- D9, the launcher at `file://` → `location.replace` → `http://127.0.0.1:port/#t`, then a write with Bearer, JSON and same Origin. Purpose met.
- D10, release, promote of a Sketch note, and draft/undraft, each through `boardCriticalSection`. Purpose met. Create gets the draft status like the CLI does.
- D11, start → lock → remove stale state → bind → write state → serve → idle exit and drain → `open` during the drain. Purpose met.
- D12, `open` from an orch session. The strand name is not a run agent's, so it proceeds with the keep-warm cap, and the server passes an empty strand name. Purpose met.
- D13, module placement. Purpose met.

**Findings:**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md:297-298` (D10 page bullet, with D10 live refresh and D8).
   - **Summary:** The record never says which revision an open edit form sends after a live refresh. The page holds R1 and opens the edit form for X. Another writer saves X, giving R2. The long poll returns the fresh projection, which carries R2 (D1 puts the revision in the projection). If the form then submits the page model's current revision (R2), D8's check passes and the operator's edit silently overwrites the other writer's change. The live refresh itself defeats the Goal's stale-write refusal.
   - **Fix:** In D10's page bullet, state that an open edit form keeps the revision of the entry as it was when the form opened, together with its unsent text, across every live refresh. Add that to the D10 Review criterion.
   - **Evidence:** `decision-record.md:8` (Goal), `:45` (revision in the projection), `:283-284` (long poll returns the fresh projection), `:297-298` (forms: only the stale-refusal display is specified).

2. **LOW, design.** `_lym/discussion/decision-record.md:146-148` (D5 start bullet).
   - **Summary:** "When the prime does not resolve, `open` returns the error `lymcwd.NamedWorktree` gives." `NamedWorktree` "spawns no git, and needs neither the worktree nor its records sibling to exist", so it does not fail when the prime is missing. The call that resolves the prime is `fabricengine.PrimeName`. A prime anchor that is gone would let the start spawn into a missing cwd and end in D5's server-log refusal, not the stated error.
   - **Fix:** Name `fabricengine.PrimeName`'s error, or a check that the prime's anchored directory exists, as the refusal before any spawn.
   - **Evidence:** `internal/lymcwd/lymcwd.go:97-112`; `internal/fabricengine/worktreelist.go:178`.

3. **LOW, design.** `_lym/discussion/decision-record.md:285` with `:511` (D10 file route) and D4's link bullet.
   - **Summary:** The file route answers not found for any name containing a separator, but a design file's name is `design_prefix` plus the slug. `checkDesignPrefix` refuses only a `decision-` prefix, so a configured prefix like `designs/` produces a design name the projection lists and the route refuses. Over `http` that link is broken.
   - **Fix:** State that the route serves exactly the names the projection lists, separators included, and refuses every other name. Or state that `design_prefix` holds no separator, and refuse one in `checkDesignPrefix`.
   - **Evidence:** `internal/boardengine/config.go:205-212`; `render.go:137`.

**Tool calls:**
- Grepped `internal/beamcli` for the `start` verb's args, the workspace folder builder and `DesignPrefix`: I needed to confirm `lym beam start root` is valid and where design file names come from.
- Read `hubworkspace.go:440-462` and `config.go:200-225`: I needed the workspace folder paths and what `checkDesignPrefix` allows.
- Ran `git log`/`git diff` on the decision record: I needed to see whether the D10 page and D5 lines are changed hunks. It returned nothing.
