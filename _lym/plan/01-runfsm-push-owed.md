# Card 1 — runfsm-push-owed

**Edit:**
- `internal/runfsm#Inputs`
- `internal/runfsm#Verdict`
- `internal/runfsm#Act`
- `internal/runfsm#Verdict.Refusal`
- `internal/runfsm#Judge`
- `internal/runfsm#Aside`
- `internal/runfsm/doc.go`
- `internal/runfsm_test#TestAct`
- `internal/battencli#TestCancelVerdict`

**Create:**
- `internal/runfsm#CodeTipGrace`

**Intent:**
Rename `Inputs.PushOwedOffline` to `Inputs.PushOwed`: true while a push of the run is owed, for an unreachable origin, a deferred records side or a held teardown lock alike (D1).
Its doc comment says so; nothing fills it until card 7.
Add `Inputs.PushOwedError string`, the owed push's last error, `Inputs.PushOwedWorktree string`, the worktree whose side is owed, and `Inputs.CodeTip string` and `Inputs.CodeTipMachine string`, the code tip the run's last transition on origin names and the machine whose transition named it.
`Act` keeps the owed push first in the rule's order, under the renamed field.
`Verdict` gains `PushOwedError`, `PushOwedWorktree`, `CodeTip` and `CodeTipMachine`, which `Act` copies from the inputs, as it copies `Owner`.
`Judge`'s `Aside.PushOwed` clears `PushOwed`, `PushOwedError` and `PushOwedWorktree`; the `Aside.PushOwed` doc names the renamed input.

`Verdict.Refusal` words the two waits anew:
- `push_owed`: `a push to origin is owed (last error: <PushOwedError>)`, way forward ``run `lym fabric push` in <PushOwedWorktree> once origin answers, then run <retry> again``.
  `lym fabric push` lands the owed side and its landing clears the failed-push record, so the re-run acts even when no process is left retrying (card 4 retires the respawns).
  With an empty error the parenthesis is left out; with an empty worktree the way forward is `run <retry> again once the owed push lands`.
  This replaces the text that origin is unreachable and its re-run once origin answers.
- `code_tip`: `the code tip <CodeTip> the run's last transition names is not on origin yet; machine <CodeTipMachine> owes it`, way forward ``run `lym fabric push` in the task worktree on machine <CodeTipMachine>, then run <retry> again once the code push lands``.
  An empty machine leaves out the clause naming it and the push clause of the way forward.

`CodeTipGrace` is the one named constant for D2's code-tip grace: a tip still missing after it, and absent from the owner's repository, is given up (card 12).
Its doc comment says what the grace measures, the time since the commit of the run's last transition on origin.

The package doc names the owed push and the code-tip wait as inputs the reader fills from tracked records and failed-push records.

`TestAct` (tier0) rewrites the `pushOwed` and `codeTip` expected texts and their rows to the new inputs:
a row with `PushOwedError` and `PushOwedWorktree` set asserts the text naming both and `lym fabric push`, a row without asserts the bare form,
the `MissingCodeTipWaits` row sets `CodeTip` and `CodeTipMachine` and asserts both named and the way forward naming `lym fabric push` on that machine,
and a row with an empty `CodeTipMachine` asserts the way forward without the push clause.
The `Judge` sub-test's `PushOwedAsideDropsTheOwedPush` row sets the renamed field.
`TestCancelVerdict`'s `HeldRunWithAnOwedPushActs` row sets `PushOwed`.

**ImpactSummary:** One renamed input field and four added fields in a pure package; the only compile-time users are two test tables, and `tendercli`'s and `battencli`'s `Aside` values keep the unchanged `Aside.PushOwed`.
