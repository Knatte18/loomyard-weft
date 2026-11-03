---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/orchrole/orch-role.md:81
    summary: The status-based test for task run versus beam reads batten's own producer, which is Run-Shed for both, so it cannot tell them apart
    fix: In the "Making a beam" section of contracts/orchrole/orch-role.md, replace the sentence that begins "`lym batten status <slug>`, filtered to its current producer, tells the two apart" with a test that works. Say that `lym batten status <slug>` reports the batten run's own rows (Worktree-Create, Seed-Child, and so on to Run-Shed), the same for a task run and a beam, so it cannot separate them. Say the two are told apart by the entry's `kind` (`lym board get <slug>`, kind project versus task) and by the inner run's current producer, read with `lym loom status` in the worktree as the stop-investigation steps of the role already do. A task run's inner producer is a tapestry, arras, darn or sampler row; a beam's is Beam-Up or a later pawl row. Check that `lym loom status` in a beam worktree reports the pawl run before writing it; if it does not, drop the producer contrast and keep only the `kind` test. Keep the next sentence, the way forward through `lym batten cancel <slug> --remove` and `lym beam create <slug>`, unchanged.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/orch-role.md:81
    summary: The new section tells the orch to read lym batten status while run-loop item 4 says never to read it on one's own
    fix: In the "Making a beam" section, tie the status read to its trigger, for example "When the operator reports that a project ran as a task run, or a stop shows it, read ..." so it falls under the exception that item 4 of The run loop already makes for a stop that needs a status read. Do not change item 4.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: contracts/orchrole/orch-role.md:70-71
    summary: The --anchor description drops two conditions the flag states, so an orch can pass a path the verb refuses
    fix: In step 2 of the section, change "a directory of the code checkout" to "a path relative to the code repo root, tracked in the commit the beam forks from", which is how `lym beam create --help` states it (internal/beamcli/cli.go:524).
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: contracts/orchrole/orch-role.md:73
    summary: The claim that only lym beam create starts a beam's run is stronger than the code, which also lets a hand batten pass and the adopt check start or resume it
    fix: Reword the sentence to "Dobby starts no project entry, so a release never makes a beam; make one with `lym beam create`." The orch has no use for the hand pass or the adopt check here, so name neither.
---

### [MEDIUM] Status read cannot tell a task run from a beam

**Location:** contracts/orchrole/orch-role.md:81
**Issue:** The section says `lym batten status <slug>`, filtered to its current producer, shows a tapestry, arras, darn or sampler row for a task run and Beam-Up or a later pawl row for a beam. The envelope carries batten's own `current_producer` (internal/battencli/arm.go:944-955, internal/shedverbs/status.go:190), and batten's rows are Worktree-Create through Run-Shed for both kinds (contracts/recipes/batten-recipe.yaml:41-84, internal/battenshed/doc.go:55). The inner run's producer is not in the envelope, except in the done-child flag. An orch following the sentence would see Run-Shed for both and could not choose between them. The board entry says only that the status "shows the rows"; the producer contrast is the writer's addition.

### [LOW] Status read against run-loop item 4

**Location:** contracts/orchrole/orch-role.md:81
**Issue:** Item 4 of The run loop says never to read `lym batten status` on your own to check on runs. The new section tells the orch to run it with no trigger, which reads as a conflict.

### [LOW] --anchor description incomplete

**Location:** contracts/orchrole/orch-role.md:70-71
**Issue:** The flag takes a path relative to the code repo root that is tracked in the commit the beam forks from (internal/beamcli/cli.go:524). "A directory of the code checkout" omits both conditions.

### [NIT] Only beam create starts a beam's run

**Location:** contracts/orchrole/orch-role.md:73
**Issue:** The code names three starters: the verb, a hand pass and the green end of the adopt check (internal/beamcli/cli.go create help, internal/beamcli/doc.go). The role says only the verb.

## Rejected

- Operator go-ahead before creating a beam (focus question): the section's "Make one on the operator's go-ahead" restates the role's working rule that a step that starts work needs an explicit go-ahead naming it; the cancel's "on the operator's word" is in the entry. Not an unlicensed addition.
- `lym fabric add` named in a section while the file header says it names no denied recovery command: `lym fabric add` is in no denied-form list (internal/shellguard/rows.go) and contracts/orchrole is outside the norecoverydeny scan (cmd/lym/norecoverydeny_test.go). The header claim holds.
- "Dobby starts no project entry": matches internal/beamcli/cli.go and doc.go.
- "tapestry, arras, darn or sampler" as the task-run inner recipes: the four recipe files exist and the role already lists them in the run loop.
- `lym batten cancel <slug> --remove`: the flag exists (internal/battencli/cancelcmd.go:130), and the role's step 3 already names it.
- `lym beam start` refuses a task pair and never makes a beam: internal/beamcli/target.go:87.
- Missing test for the new prose: static stencil prose is exempt, and an assertion on it would itself be a test-economy finding.
- Design points 1, 2, 3, 4, 5 and 6 of the entry are all covered; the section adds nothing to the existing rules the entry does not license.
