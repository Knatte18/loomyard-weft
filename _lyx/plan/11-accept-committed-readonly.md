# Card 11 — accept-audit --batch accepts a committed batch whose fabric references only read

**Edit:**
- `internal/fabricengine/refscanner.go`
- `internal/fabricengine/refscanner_test.go`
- `internal/websterengine/auditledger.go`
- `internal/websterengine/audit.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/acceptbatch_test.go`
- `internal/webstercli/acceptaudit.go`
- `internal/webstercli/acceptaudit_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Implement D14.

Classifier, in `refscanner.go` beside the reference rule: an exported function reporting whether a command is read-only.
It is when the command is built only from the readers `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq` and `cut`, each in command position, joined by `|`, `;` or `&&`, with no output redirection of any kind (`>`, `>>`, `>|`, `&>`, a numbered fd redirection, `<>`), no command or process substitution (`$(`, backtick, `<(`, `>(`) and no backgrounding (a lone `&`); separators and operators inside quoted spans are text (reuse `blankQuoted`).
The list is closed: anything else, `sort`, `uniq`, `tee`, `xargs`, `find`, an interpreter, `lyx`, `git` and `go` included, is not read-only.
The implementer audit never calls it, so a fabric-reference finding still fails the batch as correctness.

Accept route, in `auditledger.go`: `AcceptBatchFabricReference` takes the classifier as a `func(cmd string) bool` parameter.
When HEAD is the batch's start commit it accepts as today.
Otherwise it also accepts a failed batch that has commits when its recorded start commit is an ancestor of HEAD (`Git.IsAncestor`), the worktree is clean apart from the run's own state, and every Uncheckable entry is a pathless fabric reference whose recorded command is read-only.
The command is the `%q`-quoted text `fabricReferenceDetail` writes; add the inverse beside it in `audit.go`, which returns the unquoted command and false for an entry with no such text (an older record).
Refusals, each with its way forward and no denied command (`PATTERN-no-denied-recovery`): an entry that is not read-only or records no command refuses the whole call with the existing `reset --to start` then `run --fresh` steps; a start commit that is not an ancestor of HEAD and a dirty worktree refuse as today; a non-pathless entry or a batch that is not terminal failed refuse as today.
Each accepted entry is recorded as a batch audit warning, as today, its detail saying whether HEAD was the start commit or the batch's commits were kept as read-only.
State the bound in the function's comment: the evidence shows the start still lies in HEAD's history, nothing is uncommitted and no listed reader can write; it does not inspect the batch's commits; the fabric repo's own state the caller vouches for by running the verb.

`acceptaudit.go` passes the classifier and its Long help describes the committed-batch route.
`doc.go`'s paragraph on the narrow exception describes both routes.
`refusal-spec.md`: the `audit: fabric-reference` row and the `accept-audit --batch: not acceptable` row describe the committed-batch route and the new refusal, in this commit (`PATTERN-refusal-way-forward`).
Wording outside the fabric side says "fabric repo".

Tests:
- `refscanner_test.go`: a table over the classifier — each listed reader alone and in a pipeline, `;` and `&&` chains, a `|` or `>` inside quotes, against `>`, `>>`, `2>/dev/null`, `$(…)`, a backtick, `<(…)`, a trailing `&`, and `sort`, `uniq`, `tee`, `xargs`, `find`, `python3 -c`, `git` and `go`.
- `acceptbatch_test.go`: rows in `TestAcceptBatchFabricReference_Refusals` and a new acceptance test: a batch with commits, its start an ancestor of HEAD and a clean tree, whose entries are read-only, is cleared with one audit warning per entry; one entry with a redirection, with `tee`, `sort` or `uniq`, or with no recorded command refuses the whole call naming the reset-to-start and fresh-run steps and changes nothing; a start that is not an ancestor and a dirty tree refuse as before.
- `acceptaudit_test.go`: `TestAcceptAuditCmd_BatchClearsFabricReference` gains the committed read-only case through the verb.

**ImpactSummary:** the explicit accept-audit --batch route widens to committed batches with read-only fabric-reference commands; the audit itself and every other route are unchanged.
