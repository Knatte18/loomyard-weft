# Card 3 — Implementer and verify-fix bodies run gates in the foreground

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils/claims_test.go`
- `internal/websterengine/doc.go`

**Intent:**
Decision record D3 and D4: a fork ends its turn only because a gate is backgrounded, so the implementer and verify-fix bodies forbid backgrounding and run every gate in the foreground under the gate rule card 1 put in the system prompt.
They also stop telling the fork to run commands "from" an absolute path, which invites `cd <pair path> &&`.
Neither body restates the timeout value, the quick-check rule or the `cd` rule: each points at "the gate rule in your system prompt" (PATTERN-producer-pointer-rule).
Rendering is unchanged: `{{.worktree_root}}`, the card pointers and `{{.report_path}}` stay absolute and every required marker stays in the body.

`webster-body-implementer.md`:
- A new section `## Where and how you run commands`, placed after `## Prior-batch context`, reads:
  "Your session's working directory already is the worktree root, `{{.worktree_root}}`: run every Bash command from there, give the paths in Bash commands worktree-relative, and follow the gate rule in your system prompt for `cd` and for quick checks."
  "Paths you pass to the Read, Write and Edit tools stay absolute."
  "You never run a Bash call in the background: every card gate step, every card `verify:` line and the batch gate runs as a foreground Bash call under the gate rule, and you end your turn only after writing the report."
- Step 3 becomes "Run each step of the card's gate from the list below, in order, each as its own foreground Bash call under the gate rule in your system prompt." and drops the `run_in_background` sentence.
- Step 4's "run from `{{.worktree_root}}`" becomes "run from your working directory".
- Step 5's "each line of it as its own Bash call with `run_in_background`, waited for" becomes "each line of it as its own foreground Bash call under the gate rule".
- The batch gate's "each step as its own Bash call with `run_in_background`, waited for" becomes "each step as its own foreground Bash call under the gate rule".
The body then spells `run_in_background` nowhere.

`webster-body-verify-fix.md`:
- A paragraph under `## The rules`, before the bullets, reads:
  "Your session's working directory already is the worktree root, `{{.worktree_root}}`: run every Bash command from there, give the paths in Bash commands worktree-relative, and follow the gate rule in your system prompt for `cd` and for quick checks."
  "Paths you pass to the Read, Write and Edit tools stay absolute."
  "You never run a Bash call in the background, and you end your turn only after your final reply."
- The commit bullet's "from `{{.worktree_root}}`" becomes "from your working directory".
- The gate bullet becomes "Before you report, run `lyx gate test --tags integration <the findings' packages>` as a foreground Bash call under the gate rule in your system prompt, and fix what it still reports."
The body then spells `run_in_background` nowhere.

`claims_test.go`, in the `webster-body-implementer.md` entry: the row "a gate step and a card verify line run as background Bash calls…" is replaced by a `wantAll` pinning the new section heading, the working-directory sentence, the never-background sentence, step 3's and step 5's foreground wording;
a `wantNone` pins that the body names neither `run_in_background` nor ``from `{{.worktree_root}}` ``.
In the `webster-body-verify-fix.md` entry the integration-gate row is replaced by one pinning the foreground gate bullet and the working-directory and never-background sentences, beside the same `wantNone`.
The old `why` texts that say "background Bash calls" go with their rows.

`internal/websterengine/doc.go`, where it describes the card gate and the batch gate a fork runs: one sentence says the fork runs every card gate step, card verify line and the batch gate in the foreground under the gate rule in its system prompt and never backgrounds a call, so its only turn end is its final report.

**ImpactSummary:** the fork and recovery prompts and the verify-gate fixer prompt change wording only; their markers and rendering are unchanged.

**Commit:** 3: webster forks run every gate in the foreground from their working directory
