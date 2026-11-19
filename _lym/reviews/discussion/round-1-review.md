---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:105
    summary: D3's seed push cannot run before CloneAndWire as written, since CloneAndWire calls CloneHub and the board's binding is only committed inside CloneAndWire, so the stated order needs a layout branch outside fabric that the Goal forbids
    fix: "In D3, replace \"Before `CloneAndWire` runs, it pushes the seed\" with the exact place in the clone sequence where the seed is committed and pushed, and what the seed holds there (board commit with `.lym-warp` and `.lym-shortname`, prime records branch with the anchor marker). Say how `fabriccli` reaches that step without reading the layout, for example by making fabric commit and push the board seed inside `CloneHub` on every fresh bind or by a fabric verb that is the identity in a two-repository hub. Correct D3's rationale, since pushing the prime's records branch at clone is added behaviour, not the reused two-repository bind path, which pushes the board inside `CloneAndWire` and the prime's records branch only on first push."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:117
    summary: D3's refusal of an origin whose default branch tracks a root .lym-anchor applies to every one-URL clone, so it also refuses an existing two-repository weft whose board still carries the legacy hub-wide marker, and its way forward tells the operator to delete records
    fix: "In D3's fourth refusal bullet, scope the `.lym-anchor` arm so a two-repository weft never meets it: refuse only when the default branch tracks a root `.lym-anchor` and holds no `.lym-warp`, or drop the arm and keep the `lym-bolt` arm alone. Update the matching D3 acceptance criterion, and add a criterion that a one-URL clone of a two-repository weft whose board tracks the legacy `.lym-anchor` still clones."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:133
    summary: D4 deletes the records branch by its full namespaced name, which git 2.53.0 reports as [deleted] with exit 0 for a missing ref, so gitrepo's DeleteRemoteBranch loses its absent-ref answer and the destruction gate records a removal that never happened; D4's own evidence used a remote-name push that gitrepo never runs
    fix: "In D4, state how teardown tells an absent namespaced records branch from a deleted one, for example an `ls-remote` of the full ref before the delete or the leased delete on the observed tip. Replace the sentence \"git refused `git push origin --delete <branch>` under a mapped push refspec and accepted the full ref name\" with the result for the path `gitrepo` takes, a push to the origin URL: `git push <url> --delete refs/lym/heads/<missing>` exits 0 printing `remote: warning: deleting a non-existent ref` and `[deleted]`, and a short name cannot reach `refs/lym/heads/` (git 2.53.0, bare local origin). Add an acceptance criterion that a teardown of a never-pushed records branch records no remote deletion."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:96
    summary: No decision names the board branch inside refs/lym/heads/, yet it is D3's detection key and D2's remote HEAD answer; today it comes from the weft origin's HEAD and the clone's checked-out branch, neither of which a namespace has
    fix: "In D1, name the board branch and the prime's records branch under `refs/lym/heads/` and the one source both come from, preferably a fixed name constant so a renamed code default branch cannot hide the board from D3's probe. In D2, state that the records store's remote `HEAD` read returns that name. In D3, state that the one-URL `ls-remote` probes that name."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:120
    summary: D3's bound that it force-replaces a namespaced records branch only while no board branch exists is checked before anything is created, not at push time, so two concurrent --single-repo clones can leave one hub's board beside the other's records branch
    fix: "In D3, bound the seed pushes at push time: push the records branch leased on the tip, or absence, observed at the precondition check, and push the board branch leased on absence (`--force-with-lease=refs/lym/heads/<board>:`). Say a failed lease takes the refused-push teardown."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:109
    summary: A rerun after a partial seed push fetches the leftover namespaced records branch and adopts it, along with its anchor, through the weft primary's adopt path, so it is never replaced as D3 claims, and a rerun with a different --subpath is refused with no way forward
    fix: "In D3, decide whether a `--single-repo` rerun adopts a leftover namespaced records branch or replaces it, and state that one behaviour. If it replaces it, say that the records store is not fetched from, or skips adopting, a leftover branch while no board branch exists. If it adopts it, drop \"is replaced by the seed push\" and say that an explicit `--subpath` must match the leftover's anchor, naming the way forward. Align the rerun acceptance criterion."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:236
    summary: The Open risks line on the lost board body hands the parent a fact the writer could have read, since board commit 5a3ab0c956 in _bolt holds the original body (Why, Direction, To decide in Discussion) and the design covers every item it lists
    fix: "Drop the second `## Open risks` bullet. In the support log's Turn 1 and question ledger Q6, record that the original body was read from the board history at `_bolt` commit 5a3ab0c956 and that its three `To decide in Discussion` items (records branch scheme, what of f06acf39 stays, the layout probe's warnings) are answered by D1 to D4, D5 to D7 and D2."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:75
    summary: D2 limits gitrepo's namespace parameter to push, lease and push-result matching, but gitrepo also writes the upstream merge ref as refs/heads/ after a push and runs the remote delete, and D2 attributes the upstream write to fabric
    fix: "In D2, widen the `gitrepo` sentence to every `gitrepo` operation that sends or writes an origin ref name, namely push, lease, push-result matching, the upstream `branch.<b>.merge` write after a push, and the remote deletes, plain and leased. Change \"the upstream config fabric writes\" to \"the upstream config fabric or `gitrepo` writes\"."
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:46
    summary: D1's git evidence is a scratch-directory experiment with results described but no recorded commands and output, test or documentation pin, so every later round must rerun it
    fix: "In D1, pin each git fact the design rests on: cite git's documentation, git-fetch(1) and git-push(1) on refspec mapping and on an explicit refspec bypassing `remote.<name>.push`, and gitglossary on `--force-with-lease` with a remote-tracking ref. Or record the exact commands and their output lines, with git 2.53.0 named. Say that the D1 and D4 acceptance criteria pin them as tests over `hubforge`'s bare origin."
  - id: F10
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:107
    summary: D3's teardown names only a seed push that origin refuses, so the hub directory's fate after a transport failure between the records push and the board push is unstated
    fix: "In D3, change \"A seed push that origin refuses\" to \"A seed push that fails, refused or unreachable,\" so every failed seed push takes the same teardown and names its cause."
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:197
    summary: D9 lists the PATTERN backgrounds that lose their one-repository text but omits PATTERN-cwd-resolution's and PATTERN-run-fsm's, which both carry one-repository paragraphs that D5 and the PATTERN changes make false
    fix: "In D9, add `PATTERN-cwd-resolution` (the one-repository `ResolveWithin` line and the worktree-root anchor clause) and `PATTERN-run-fsm` (the `operator_unpushed` and `protected` paragraph and its `TestPrimePushGuard` pin) to the backgrounds that lose their one-repository text."
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:180
    summary: D8's builder drives D3's clone by one URL over a bare origin with no namespace, which D3 sends down the two-repository path, so the builder needs --single-repo
    fix: "In D8, say the builder seeds the origin with `lym fabric clone --single-repo <url>` and builds a second machine's hub with a plain one-URL clone."
  - id: F13
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:285
    summary: The plan-writer note lists originreconcile.go, cleanupremotewarp.go, remoteleftover.go, archive.go and recordsprobe.go after internal/gitrepo/push.go, but those files live in internal/fabricengine
    fix: "In `## Notes for the plan writer`, split the list by package: `internal/gitrepo/` `push.go`, `pushretry.go`, `pushdestination.go`, and `internal/fabricengine/` `originreconcile.go`, `cleanupremotewarp.go`, `remoteleftover.go`, `archive.go`, `recordsprobe.go`."
---

### [BLOCKING] D3's seed push has no place before `CloneAndWire`

**Location:** `_lym/discussion/decision-record.md:105`

**Issue:** `CloneAndWire` is the entry point, and it calls `CloneHub` (`internal/fabriccli/clone.go:37`), so nothing can run before it.
On a fresh bind, the end of `CloneHub` leaves the board as an unborn orphan branch (`internal/fabricengine/boardweft.go:55-57`).
`.lym-warp` and `.lym-shortname` are written to disk and committed only by the CLI-layer Bolt commit (`internal/fabricengine/clone.go:395-418`, `internal/fabriccli/clone.go:74`).
The two-repository path D3 claims to reuse pushes the board inside `CloneAndWire` (`internal/fabriccli/clone.go:92-96`), and leaves the prime's records branch to its first push (`internal/fabricengine/clone.go:513`).
To push records first, board last and before the wiring, D3 needs a layout-aware step in `fabriccli`.
That step is what f06acf39's `PushOneRepositoryClone` was, which D5 removes and the Goal forbids.
Interactions F4 found the same contradiction.

### [BLOCKING] D3's `.lym-anchor` refusal catches a two-repository weft

**Location:** `_lym/discussion/decision-record.md:117`

**Issue:** The refusal runs on every one-URL clone.
A two-repository weft's default branch is its board, and a hub that was never reconciled still tracks the legacy hub-wide `.lym-anchor` at the board's root.
`CloneHub` still adopts that marker (step 5 of its doc comment), and `retireBoltAnchorMarker` (`internal/fabricengine/anchormarker.go:186`) removes it only on reconcile.
Such a clone is refused, contradicting Scope Out's "no behaviour change to a two-repository hub".
The way forward then tells the operator to remove `.lym-anchor` and `_lym` from what are the records branches.

### [MEDIUM] D4's full-name delete reports a missing ref as deleted

**Location:** `_lym/discussion/decision-record.md:133`

**Issue:** `gitrepo.Repo.DeleteRemoteBranch` deletes by short name on purpose (`internal/gitrepo/push.go:313`).
The short name lets it return `(false, nil)` for an absent ref, which the destruction gate records (`internal/fabricengine/destroy.go:1631,1643`).
Against a bare local origin, git 2.53.0 gave:
- `git push <url> --delete refs/lym/heads/missing-weft` exited 0, printing `remote: warning: deleting a non-existent ref` and `- [deleted]`;
- `git push <url> --delete x-weft` for an existing `refs/lym/heads/x-weft` answered `remote ref does not exist`.

The full name is therefore forced, and with it a false deletion.
D4's evidence pushed to the remote name under a push refspec, a path `gitrepo` never takes, since it pushes to `dest.url`.
The interactions, walkthrough and bound-breaker lenses found the same defect.

### [MEDIUM] The board branch's name in the namespace is never decided

**Location:** `_lym/discussion/decision-record.md:96`

**Issue:** Today the board is the weft's default branch.
`remoteDefaultBranch` reads it from origin's `HEAD` and parses only `ref: refs/heads/` (`internal/fabricengine/cleanupremotewarp.go:308-325`), and the clone takes it from the checked-out branch (`checkedOutBranch`).
A records store made by init and fetch has neither.
D3's detection `ls-remote` needs the name before any store exists, and D2's remote `HEAD` answer needs it too.
Derived from the code default branch, a rename of that branch hides the board, and `--single-repo` then seeds a second hub in the same namespace.
All four lenses found this.

### [MEDIUM] D3's force-replace bound is checked too early

**Location:** `_lym/discussion/decision-record.md:120`

**Issue:** "While no board branch exists" is checked at the start of the clone.
Two `--single-repo` clones interleave as follows:
1. Clone A pushes its records branch.
2. Clone B passes its precondition and force-replaces A's records branch.
3. A's board push lands.
4. B's board push is refused, and B tears down.

Origin is left with A's board beside B's records.

### [MEDIUM] D3's rerun adopts the leftover instead of replacing it

**Location:** `_lym/discussion/decision-record.md:109`

**Issue:** D1's fetch refspec brings the leftover branch in as `origin/<prime>-weft`.
`suffixWeftPrimaryBranch` checks it out (`internal/fabricengine/clone.go:515-524`), and the anchor is adopted from it.
With a different explicit `--subpath`, `adoptRecordedAnchor` refuses with no way forward (`internal/fabricengine/clone.go:478-479`).
The force-replace D3 describes is never reached.

### [MEDIUM] An Open risks line hands the parent a researchable fact

**Location:** `_lym/discussion/decision-record.md:236`

**Issue:** `git -C _bolt show 5a3ab0c956:design-single-repo-records-branches.md` prints the body as the operator wrote it, and `b8e9afe900` is the board sync that replaced its opening with the error object.
Its `To decide in Discussion` items are covered by D1 to D7.
Per the rubric's deferred-research bullet, this is the writer's fact to settle, not a question for the parent.

### [LOW] D2's `gitrepo` parameter list misses the upstream write and the deletes

**Location:** `_lym/discussion/decision-record.md:75`

**Issue:** `recordPush` writes `branch.<b>.merge` as `refs/heads/<b>` (`internal/gitrepo/pushdestination.go:114`).
`DeleteRemoteBranchLeased` hard-codes a `refs/heads/` lease (`internal/gitrepo/push.go:345`).
D2's own bullet requires the namespaced upstream, so the gap is between D2's two statements, and its acceptance criterion would catch it.

### [LOW] D1's git facts are not pinned

**Location:** `_lym/discussion/decision-record.md:46`

**Issue:** D1 describes the results of a scratch experiment and records neither the commands with their output nor a test or doc pin.
The GitHub acceptance fact is pinned well enough: it cites a mailing-list thread and GitHub's docs, and D3's refusal bounds a host that rejects the push.

### [LOW] D3's teardown covers a refused push only

**Location:** `_lym/discussion/decision-record.md:107`

**Issue:** A transport failure between the two seed pushes falls outside "a seed push that origin refuses".

### [LOW] D9's list of PATTERN backgrounds is incomplete

**Location:** `_lym/discussion/decision-record.md:197`

**Issue:** `pattern/PATTERN-cwd-resolution.md` keeps the one-repository `ResolveWithin` line and the worktree-root anchor clause.
`pattern/PATTERN-run-fsm.md` keeps the `operator_unpushed` and `protected` paragraph.
D9's named list omits both backgrounds; only the D9 `Review:` criterion would catch them.

### [NIT] D8 omits `--single-repo`

**Location:** `_lym/discussion/decision-record.md:180`

**Issue:** By D3's own rule, a one-URL clone of a bare origin with no namespace takes the two-repository path.

### [NIT] The plan-writer note puts fabric files under `internal/gitrepo`

**Location:** `_lym/discussion/decision-record.md:285`

**Issue:** Five of the eight listed files live in `internal/fabricengine`.

## Rejected

- Open risks on the peer run `two-repo-merge-probe-regression`: no finding. It is a design conflict with another running run, which is `decision` class for the parent, and D7's conditional covers either landing order.
- Open risks on the stale title of project `single-repo-hub`: no finding. The parent entry is contradicted by this design, which is the parent's call.
- Walkthrough's D2 note that `--track` derives the upstream through the fetch refspec: superseded. `recordPush` writes `refs/heads/<b>` explicitly (`internal/gitrepo/pushdestination.go:114`), which F8 covers.
- Relocation: no `decision-record.md` content was found to be deliberation that belongs in the support log. D1's evidence paragraph is the pin the rubric asks for.

## Focus departures

None.
