# Card 13 — pr-continue-autonomy

**Create:**
- `plan:internal/loomcli#prContinueReview` -> `type prContinueReview struct`

**Edit:**
- `internal/loomcli/prcontinue.go`
- `internal/loomcli#TestPRContinueVerb`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/landingshed#ReadPRReviewStanding`
- `plan:internal/landingshed#PRReviewSatisfied`
- `plan:internal/landingshed#PRReviewAwaiting`
- `internal/loomcli#pullRequestDeps`
- `internal/pullrequest#Record`

**Intent:** D4: under a review's recorded autonomy, `pr-continue` sets the continue flag only on the parent's satisfied answer at the current head.

The card changes three parts of `prcontinue.go`: its file header, `prContinueVerb` and `loomCLI.prContinueCmd`.
`prContinueVerb` takes the review as a fourth parameter: `func prContinueVerb(out io.Writer, recordPath, lockPath string, review prContinueReview) int`.
The file header, which says the agent runs the verb to tell the run the operator is satisfied,
says that under a review's recorded autonomy the verb ends the review only on the parent's recorded satisfied answer at the current head.

`prContinueReview` carries `head func() (string, error)`, `standing func(head string) (landingshed.PRReviewStanding, error)`
and `closed func(rec pullrequest.Record) (bool, error)`, which reads the pull request's state on GitHub.
`prContinueCmd` builds it over `c.location`: the head from `fabricengine.Open`'s `HeadSHA`,
the standing from `landingshed.ReadPRReviewStanding(burlerengine.ParentReviewFilesOf(c.location.AnchorPath()), head, time.Now())`,
and `closed` from `pullRequestDeps(c.location, rec.Base, false, nil)`'s `Origin` and `API`, then `Status`.

`prContinueVerb`, on a record whose autonomy is recorded true and that is not `ReviewEnded`:

1. a closed pull request is refused, its way forward naming the row's closed-without-merge halt:
   leave the pull request alone and end the turn, since the PR-Review row halts on its next wake and the parent settles that halt;
2. a standing of `PRReviewSatisfied` at the current head sets `Continue` and clears `MailedHead`, as today's success;
3. otherwise, with a mailed head standing, it clears `MailedHead` only, so the row gates that head again, and answers ok with `continue` false and a note that the review stays open;
4. otherwise it refuses.

The note of case 3 and the refusal of case 4 name waiting for the parent's answer with `lym loom parent-review await` when the standing is `PRReviewAwaiting`,
and asking again with `lym loom parent-review ask` otherwise.
A read failure of the head, the standing or the state is an error envelope with today's retry way forward.
A record whose autonomy is false or absent runs as today, so `pr-continue` succeeds with no parent answer.
The Long help and `doc.go` say so.

`TestPRContinueVerb` (tier0) calls the verb with a stub `prContinueReview`; its existing rows pass a review whose record has no autonomy and keep their expectations.
New rows with autonomy true:

- no satisfied answer and no mailed head: refused, naming `lym loom parent-review ask`;
- the newest ask awaiting at the head: refused, naming `lym loom parent-review await`;
- satisfied at an older head (standing `ask-again`): refused;
- satisfied at the current head: `Continue` set, `MailedHead` cleared;
- a mailed head with no satisfied answer: `MailedHead` cleared, `Continue` unset, ok with the stays-open note;
- a closed pull request: refused, naming the row's halt and that the parent settles it, and naming neither `lym loom resume` nor `lym batten cancel`.

The toggle criterion holds by construction: the verb takes no autonomy seam and reads only the record's value.

`TestPRContinueVerb` also gains one row over the real reading: its `standing` is `landingshed.ReadPRReviewStanding` over a temp ask record
whose newest PR-Review ask at the current head was answered satisfied through `burlerengine.AnswerParentAsk`, after an earlier Code-Review ask;
`Continue` is set, so the answer the parent records ends the review (D4's answer-then-continue criterion).

**ImpactSummary:** `prContinueVerb` gains one parameter whose two callers, its command and its test, are on this card; behaviour changes only for a record holding autonomy true.
