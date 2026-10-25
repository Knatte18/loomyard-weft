```yaml
lens: way-forward
outcome: reported
```

Task: the way-forward lens. For each refusal, stop, reset or recovery step a card adds, check that its named next step can succeed from the state it leaves.

**Per refusal: the next step, and whether it progresses**

- Card 1, invalid own marker: an error naming the file, with no next step. The next step is unnamed.
- Card 2, `dobbyRuns` on an invalid own marker: an error naming the pair and `ErrInvalidAnchor`, with no next step. The next step is unnamed.
- Card 7, `refuseEmptyOwnMarker`: `git -C <weft> restore .lyx-anchor`, or write the anchor and commit it, then re-run. Progresses.
- Card 8, reconcile on a pair with an empty marker: the same restore or commit, then reconcile again. Progresses.
- Card 9, pull stop: `lyx fabric anchor <slug> --set <wantAnchor>`. If the warp lacks `B`, the `--set` names a warp update by hand first. Progresses.
- Card 9, replaying-push stop (`anchor_change` record): the `--set`, after which `Finish` spawns the push. Progresses.
- Card 9, push on an empty marker: restore the marker file. Progresses.
- Card 10, checkout refusal: the `--set`, then the checkout again, with a warp update first when needed. Progresses; the cwd gate redirects the operator to the new anchor.
- Card 10, drop-local refusal: `--set <origin's anchor>`, then `lyx adopt <slug> --drop-local`. Progresses only when the pair has no unfinished run (Finding 1).
- Card 11, `anchor_stopped` on a pair: the `--set`. Progresses only when the pair is owned here, or the `--set` targets the upstream's marker (Finding 1).
- Card 11, `anchor_stopped` on the prime: `lyx reed down` everywhere, wait for batten runs, the `--set`, then `lyx orch start`. Progresses; the `--set`'s own refusal names `lyx batten pause` for runs that never halt.
- Card 11, `judgeOrigin` letting `lyx batten cancel` through: the cancel, then the `--set`. Progresses only when the slug is owned here (Finding 1).
- Card 12, promote refusal: `--set <parent>`, then `lyx beam promote <slug>`, whose earlier steps are skipped on the re-run. Progresses.
- Card 13, `MoveInProgress` with the lock held: wait, then `--set <B>`. Progresses.
- Card 13, `MoveInProgress` with the lock free: the `--set <B>` re-run, which skips the owner refusal. Progresses.
- Card 13, the record-standing stops in Add, reconcile, pull, push, checkout and the origin reconcile: the same text. Progresses.
- Card 15, empty own marker: restore the marker. Progresses.
- Card 15, no committed marker: `lyx fabric reconcile`. Progresses.
- Card 15, dirty weft: `lyx fabric commit`, remove the files, or `git restore .lyx-anchor`. Progresses.
- Card 15, a `B` the warp lacks while the warp's upstream tracks it: update the warp by hand. Progresses.
- Card 15, `B` fails `ValidateAnchorRel`, is untracked everywhere, crosses a link or reserved name, or holds a real wired name: no next step is named (Finding 3).
- Card 15, a re-run naming another `B`: the record's `B`. Progresses.
- Card 15, both `A/.lyx` and `B/.lyx` exist: merge by hand, remove `B/.lyx`, re-run. Progresses.
- Card 15, a failure before the commit: everything is restored and the record removed; the CLI restarts the dobbies. Progresses.
- Card 15, an `Apply` failure after the commit (steps 5–7): no next step is named, and the dobby restart cannot succeed (Finding 2).
- Card 14, integration conflict: tag, save, `reset --keep <commit>^`, re-run, write the changes back. Progresses; the weft stays clean, so the first-run dirt check passes.
- Card 16, `EnsureLive` refused: the `MoveInProgress` text. Progresses.
- Card 17, the board names another owner: `lyx adopt <slug>`, or run the verb on the owner. Does not progress when the weft is `anchor_stopped` (Finding 1).
- Card 17, unfinished run: wait, or `lyx batten cancel <slug>`. Does not progress for a slug owned elsewhere (Finding 1).
- Card 17, a live strand: `lyx reed down`. Progresses.
- Card 17, a live dobby on Windows: `taskkill`, then the `--set`. Progresses.
- Card 17, the prime's hub-wide refusals: `lyx reed down`, `lyx batten pause` or wait, wait for the gate slot holder, `lyx hibernate cancel`. Progresses.
- Card 17, `<slug>` not present on this machine: `lyx fabric pairs`. Progresses.
- Card 18, `EnsureAnchor` with a different anchor: the `--set`, then `lyx beam create <slug> --anchor <anchor>`. Progresses.
- Card 18, an invalid, untracked or link-crossing `--anchor`: no next step is named (Finding 3).
- Card 19, adopt during a move: the `--set <B>`, which skips the owner refusal while the record stands, then `lyx adopt` again. Progresses.

**Findings**

1. MEDIUM, class design, cards 17, 11 and 10.
   - **Summary:** for a slug owned by another machine, the way-forward chain loops.
     - Card 17's owner refusal (line 43) names `lyx adopt <slug>`.
     - Card 17's unfinished-run refusal (line 45) names `lyx batten cancel <slug>`.
     - Cancel refuses an owned-elsewhere slug and names `lyx adopt`.
     - Adopt stops on the slug's `anchor_stopped` weft (card 11, line 37: `stoppedBranches` stops the owner write). Its drop-local refuses on the marker mismatch (card 10, line 33). Both name the `--set`, which is refused again.
   - **Concrete case:** this machine has an unpushed local re-anchor, and the owner pushed a different one. The stop names the local tip's anchor (card 9's `wantAnchor` rule), which the owner refusal blocks.
   - **The step that works is never named:** `--set <the upstream's marker>`, which D7 leaves open to every machine.
   - **Evidence:** `internal/battencli/cancel.go:389-390` and `:424` (`cancelOwnedElsewhere` names `lyx adopt`); `_lyx/plan/17-fabric-anchor-verb.md:43,45`; `_lyx/plan/11-origin-reconcile-stops-on-marker-change.md:37`; `_lyx/plan/10-checkout-and-drop-local-refuse-another-anchor.md:33`.
   - **Fix:** the owner refusal in card 17, and the unfinished-run refusal for an owned-elsewhere slug, name `lyx fabric anchor <slug> --set <the fetched upstream tip's marker>` as an exit that is always open.

2. MEDIUM, class design, cards 15 and 17.
   - **Summary:** an `Apply` failure after the move commit (wiring at `B`, launchers, workspace regeneration) has no prescribed end state or way forward.
     - Card 15's restore covers only failures before the commit (line 51). After the commit, the record stands.
     - Card 17's `Apply`-error path "restarts every dobby it ended" (line 53). Card 16's `ensureOne` gate (line 36) refuses every such start while the record stands, so the restart cannot succeed.
     - The returned error is not said to name the landed re-run, `lyx fabric anchor <slug> --set <B>`.
   - **Evidence:** `_lyx/plan/15-anchor-change-engine.md:51`, `_lyx/plan/17-fabric-anchor-verb.md:53`, `_lyx/plan/16-dobbies-wait-out-a-move.md:36`.
   - **Fix:** an `Apply` error after the commit leaves the record, names the `--set <B>` re-run, and the CLI skips the dobby restart, which the re-run performs.

3. MEDIUM, class consistency, cards 15 and 18.
   - **Summary:** the `B` validation refusals name no way forward, against the record's constraint "Every refusal names its way forward".
     - Card 15 (line 42) names one only for the case where the warp's upstream tracks `B`.
     - Card 15 names none when `B` fails `ValidateAnchorRel`, is untracked in the warp and its upstream, has a link, wired, `.lyx`, `.scratch` or hub-reserved segment, or holds a real wired name (the warp-pristine case).
     - Card 18's `--anchor` validation refusal (line 34) names none either.
   - **Evidence:** `_lyx/plan/15-anchor-change-engine.md:42`, `_lyx/plan/18-beam-create-at-an-anchor.md:34`, decision record Constraints ("Every refusal names its way forward").
   - **Fix:** each of these refusals names its next step: another directory, committing the directory in the warp, or moving the real wired name out of `B`.

4. LOW, class consistency, cards 1 and 2.
   - **Summary:** an invalid own marker, and the `dobbyRuns` error built on it, name the file but no next step. Every resolve in that worktree then fails.
   - **Missing next step:** correct `<hub>/<name>-weft/.lyx-anchor` and commit it with `git -C`, since `lyx fabric commit` does not stage the marker (D7).
   - **Evidence:** `_lyx/plan/01-lyxcwd-own-marker.md:30`, `_lyx/plan/02-location-literal-scan.md:23,35`.

Tool calls:
1. Grepped the `pairs`, `pause` and `cancel` verb definitions and `battencli` cancel's owner check, to confirm the named verbs exist and whether `lyx batten cancel` refuses an owned-elsewhere slug. The manifest held no `battencli` cancel code.
