---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/00-overview.md:42
    summary: Report-head and last-batch-head sites name plain git reset --keep, which D7 does not license; the plan overrides D7 in its own Shared Decisions
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/15-webster-no-denied-recovery.md
    summary: Card 15's Intent does not state what the never-marker exemption lets through or what bounds it, and its self-test does not pin git reset --keep and git checkout as outside the denied set
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/04-claude-bash-stdin-hook.md
    summary: The smoke test that pins D3's allow/deny-unchanged requirement skips on the operator's settings and is never run by verify, so the denied half of the pin and the compound-command matching risk are left to chance
---

### [MEDIUM] Report-head sites name `git reset --keep`, a choice D7 does not grant

**Location:** `_lyx/plan/00-overview.md` (Shared Decisions, "the report-head and last-batch-head moves"), `_lyx/plan/15-webster-no-denied-recovery.md`, `_lyx/plan/12-webster-reset-verb.md` (standalone refusal).
**Issue:** D7 says every refusal, spec row and stencil sentence that says "move HEAD back … with git" names `lyx webster reset --to start` or `--to pre-fix` instead.
D6 and A6 bound the verb to those two targets, so D7 and D6 cannot both be met for the `head_sha` and last-batch-head sites.
The plan resolves the conflict by naming `git reset --keep <sha>` there, and in card 12's standalone refusal.
The command is not denied, so the goal behind D7 holds.
The record does not grant that substitute, though, and the plan presents it as a settled decision rather than a departure for the operator to confirm.
**Fix:** In the overview decision and in cards 12 and 15, state it as an explicit departure from D7 that needs operator confirmation, with the reason (A6's premise that every Webster refusal needs one of the two targets does not hold for the `head_sha` sites), and state that D7's verb rule is carried for every `start` and `pre-fix` site.

### [LOW] Card 15's exemption marker has no stated bound

**Location:** `_lyx/plan/15-webster-no-denied-recovery.md`, D10 paragraph.
**Issue:** The scan's `never ` marker decides what passes.
The Intent says the exemption applies only when the marker is immediately before the form, but not what that lets through (a denied form inside a sentence that otherwise offers it, once the marker sits right before it) or what bounds it.
The self-test plants `git reset --hard` as a way forward and "never `git reset --hard`" as a prohibition, but never pins that `git reset --keep <sha>` and `git checkout <head> -- <path>` stay outside the denied set, the property F1's choice and D7's last sentence rely on.
It is also silent on whether a longer form such as `git push --force-with-lease` matches `git push --force`.
**Fix:** Add to the Intent the exemption's reach and bound (one marker, immediately before the form, a prohibition only; a way forward after a marker still fails because the marker must directly precede the form), a self-test planting `git reset --keep <sha>` and `git checkout <head> -- <path>` as passing, and a matching rule for the forms (a form matches only as a whole command token sequence).

### [LOW] Card 4's smoke test cannot pin D3's matching requirement by itself

**Location:** `_lyx/plan/04-claude-bash-stdin-hook.md`, Tests.
**Issue:** D3 requires a test that pins unchanged allow and deny matching for an allowed and a denied command.
The smoke test skips when the operator's Claude settings carry no `git reset --hard` deny rule, so the denied half is pinned only on machines that happen to have it.
`verify:` only compiles the `smoke` tag, so nothing runs it by default.
The Intent also does not name the risk that makes the pin matter: the rewritten command is `exec </dev/null; <command>`, and a permission matcher that splits compound commands sees `exec </dev/null` as a subcommand no allow rule covers.
**Fix:** Have the smoke test install its own project-level deny rule for `git reset --hard` and allow rule for `git status` in the hub, so it does not depend on the operator's settings and skips only when the `claude` binary is absent, and name the compound-command risk and the D3 fallback trigger in the Intent.

## Notes on the focus questions (no finding)

- Cards 3 and 5 carry D1's declared-prefix carve-out and the per-tick expiry check, and the expiry's `OutcomeAsking` is reached only after `allOutputFilesExist` is consulted.
- Cards 13 and 14 carry D5's absent-file re-entry (verified in `Run`, which archives the stale outcome at entry and starts a fresh Master) and D8's single re-entry step, and card 14 gives `AcceptPendingAudit` per-finding reasons.
- Card 15's size and the 11/12 split are not defects: the scan must land with the commit that completes D7, and the plan-versus-verb split gives each card an independently testable unit.
- `verify:` runs `go test ./...` and `-tags integration ./...` over every package any card targets and vets the `smoke` tag, so it covers `cmd/lyx`, `internal/webstercli`, `internal/shuttlecli`, `internal/fabricengine` and `claudeengine`.
- Card 8 checks containment, ownership, dirtiness and force in that order with force always false, and states the own-path exemption's bound.
