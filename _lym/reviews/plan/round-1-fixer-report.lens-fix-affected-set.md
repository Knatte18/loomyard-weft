```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-based finding's search and check that the fix covers every member.

Unchecked findings: none.

no findings

How each set check came out:
- **F1 (sites that move the run through loom's guard):** loomcli builds exactly two `RelaunchRequest`s, at `arbiterconsume.go:21` and `drivewiring.go:274`. Its one `RunGoto` is at `drivewiring.go:332`, over `c.spec`, which is armed from `specFor` (`cli.go:246`, `arm.go:296`). Card 9 wraps all three. The verdict-check guards at `drivewiring.go:208` and `arbiter.go:140` stay bare, as the card says.
- **F2 (docs that list runproc's role set):** `internal/runproc/doc.go:4` is the only place that lists it, and card 1 now edits that file.
- **F3 (comments naming the deleted webster members):** the parent's earlier grep is in context and needs no rerun. Card 3 now covers `render.go`, `strand.go`, `recoverbatch.go`, `template_test.go:375`, `doc.go:423` and `doc.go:560`, the `ClassifyInputs.TurnEnded` comment and the `WebsterBodyVerifyFix` doc comment. Card 3 already handled `webster-body-verify-fix.md:2` and the stencils embed, registry and marker rows. The comments on `template_test.go:378` and `stencils.go:286` go away with the symbols they describe. The `classify_test.go` comments name the field `StrandLive`, which stays. The `docs/` hits are measurement reports, which may go stale.
- **F4 (text that describes the gate element's keys):** the spec is now on card 7. `entries_gate.go:20` and `:43` describe only what `resolveGateEntry` checks. Card 7 strips `fixer` before that check runs, so both stay true. `entries_parts.go` and `shuttleengine/wait.go` describe other things.

Calls:
1. grep over loomcli for request, `RunGoto` and spec construction sites, docs listing runproc roles, and files mentioning `pass_on_cap`: my context held the earlier, wider greps but not these exact sets.
2. grep for `pass_on_cap` in the `shedrecipe` sources: to check whether the gate element comments go stale, which my context had not shown.
