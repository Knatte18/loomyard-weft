```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, notice, next step and criterion the fix added can be reached and moves things forward.

Ways forward the fix introduced:
1. D3 Bound and D3 criterion: off the owner's machine, the create is refused naming the owner and `lyx adopt <slug>`.
2. D8: in a beam whose entry is run-held by its landing, the guard refuses naming the landing; D8 criterion.
3. D11: the start guard's left-pair refusal gives way for a marked beam's landing; D11 criterion.
4. D13: when the beam's worktree is absent on this machine, the ready notice goes to the prime's orch.
5. D7 criterion: a beam member starts while the prime holds tracked dirt.

Items 1 and 5 hold. Item 1 works because D8 lets `lyx adopt` take over a beam project's owner without a pair, and D9 then admits the create here. Item 5 works because D7's start guard checks no dirt for a pair forked from a beam.

Findings:

- **MEDIUM** — D8, "the guard refuses in a beam whose board entry is run-held by its landing, naming the landing".
  The refusal names no place to run the verb, so it leaves the operator stuck.
  The operator's sessions in the beam during its landing are refused too.
  To see, resume or cancel a halted landing (`lyx batten status`, `run`, `cancel <slug>`), the verb has to run from the beam's parent worktree or the prime, and the refusal names neither.
  D8's other guard refusal does name the nearest beam or the prime, and PATTERN-refusal-way-forward needs a step that can be carried out.

- **MEDIUM** — D13, "A beam whose worktree is absent on this machine sends the notice to the prime's orch".
  The notice goes to an orch that is told never to act on it.
  An unspawned task has no origin record, so its `parent_beam` (D4: `ParentOf` falling back to `ParentBeam`) is the absent beam.
  D8's stencil filter tells the prime's orch to start only entries with an empty `parent_beam`.
  Even an attempted start would stop at Worktree-Create (D7: `lyx adopt <beam>` or `lyx beam create`), or be refused under the beam's owner on another machine.
  The notice should name the beam's owner machine and `lyx adopt <beam>`, or be left out.

- **LOW** — D11 criterion, "passes batten's start guard and reaches Worktree-Create's skip".
  The exemption covers only the left-pair refusal.
  With no status file, the start guard also refuses on the prime's tracked dirt (`internal/battencli/startguard.go:67-75`), so a beam landing with a dirty prime never passes, even when the beam's parent is another beam.
  D7's "keeps its tracked-dirt refusal for a pair forked from the prime" doesn't say which way a landing goes, since it forks no pair.

- **LOW** — D11, the exemption needs the local pair to carry "the beam mark in its origin record".
  When the beam's records worktree is missing, `ReadOriginFor` finds no record (`internal/fabricengine/origin.go:121-126`).
  So the left-pair refusal still fires and names `lyx batten cancel <slug> --remove`, which removes the beam.
  The way forward for a beam in that state should be `lyx fabric reconcile`, or `lyx adopt` of the beam.

Outside my lens: D4's "read as the prime only when that names the prime's worktree" compares a worktree name recorded on another machine, whose prime directory name may differ from this machine's. That is a question for fix-claims.

Tool calls:
- Read `internal/battencli/startguard.go:30-62` and `cancelmarker.go:166`, to check whether the start guard's cancel-marker and dirt checks still block the exempted landing; my context showed only lines 60-100.
