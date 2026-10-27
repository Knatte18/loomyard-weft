# Ideas from r/ClaudeCode "one line in your CLAUDE.md": fit for LoomYard and bench measures

`claude-md-thread-ideas` · Note · research, stencil, loom, burler

# r/ClaudeCode "What's the one line in your CLAUDE.md that made the biggest difference?" — review for LoomYard

Status: the operator decides whether a research task with a bench is drafted from section 1; nothing is promoted yet (2026-10-26).

Source: the r/ClaudeCode thread named in the title, read from a 17-page PDF export kept only on OSL-1046.
The post shows 887 votes and 187 comments.
The export shows no per-comment vote counts, so "prominence" below means position under Best sort, reply count, or the OP's own endorsement.
Paths below are repository-relative unless they start with `~/.claude`, which stands for `/home/AD.NORCERESEARCH.NO/hanf/.claude/plugins/cache/scribe/scribe/1.1.0/skills`.

## 1. Ideas most worth a bench, ranked

1. **Trust the code for what happens, and report every doc/code disagreement** (SirRobinBlibs, cites arXiv 2610.30571).
   Measure: seed a fixture hub with stale statements in `CLAUDE.md`, a `doc.go` and a `pattern/` file, run darn on tasks that touch them, and count the share of seeded conflicts the change description or a friction note reports, and the share where the agent copied the stale text.
2. **Never weaken, skip or delete a test or assertion to make a change pass** (NamerekDev, namrog84).
   Measure: per run, a diff scan of the task branch for removed `t.Run` cases, removed assertions, added `t.Skip` and deleted test functions, plus a planted task whose correct fix needs a test that is hard to satisfy.
3. **Read every file you need in one pass, then edit, then test once** (the post itself, 887 votes).
   Measure: on darn runs, token spend per run, `lym gate test` calls per run, Reads issued after the first Edit, and Code-Review findings of class `consistency` naming a missed sibling site.
4. **Before writing "we do X" or "we don't have X", reopen the code and cite `file:line`** (Stunning-Sherbet1853; OP said "stealing it").
   Measure: sample the factual claims about the codebase in change descriptions, decision records and board notes, check each against the code, and count false claims per run.
5. **Mutation check after writing tests** (namrog84: break one behavior on purpose and confirm a test fails).
   Measure: mutation score of the tests a run added or changed, computed over the changed packages by a Go mutation tool, compared across arms.
6. **A mistake names its instruction cause** (pixxelkick, adapted by Classic-Asparagus: "a missing, ambiguous, or conflicting instruction, or a clear instruction you did not follow").
   Measure: share of friction-filed issues that name a specific stencil or rule line as the cause, and the recurrence rate of the same issue in later runs after the cited line is fixed.
7. **Read the related tests before touching anything** (masiha97).
   Measure: first-gate failure rate per card or per darn commit, and self-fix attempts spent per card.
8. **Cheap model for the fresh-eyes reviewer** (pixxelkick: "spin up a sub agent with minimal context using a cheap model").
   Measure: run Code-Burler with the reviewer on a smaller model against the default, and compare findings per run, the fixer's dispute rate (false positives) and findings a later round or landing catches (false negatives), against token cost.

## 2. Every idea

### Read all, edit all, test once (post, 887 votes)

"Before changing anything, list every file you'll need and read them in one pass. Then make all edits, then run the tests once."
A reply (page 15) adds: keep one test run before the edits, so a failure at the end is attributable.

- Verdict: partly.
- Have: webster's FRESH-READ rule, `contracts/stencils/webster/webster-body-implementer.md:5-7`, and span reads via `lym quarry resolve` at `:53`; burler's pre-read for fork reviewers, `contracts/stencils/burler/burler-fork-reading.md:9-16`; twill's writer fork, `contracts/stencils/twill/twill-writer-fork.md:9`.
  The baseline half exists as webster's "a failure the card did not cause" test, `webster-body-implementer.md:113-122`, and drift's baseline, `contracts/stencils/drift/drift-template-baseline.md`.
- Missing: the darn writer gets no read-first rule and is told "Test as you work", `contracts/stencils/darn/darn-template-write.md:72-74`, the opposite of "test once".
- Where: `darn-template-write.md` Step 4.
- Bench: yes, see rank 3.

### Treat corrections as a queue, do not drop current work (pixxelkick, top comment)

- Verdict: does not fit as written; the orch analogue is already in place.
- Why: spawned agents take no mid-run operator corrections except recorded decisions (`lym loom decision add`, `.lym/orch/role.md:284`), and the orch puts every proposal "on the board as a note at once, never held in your head", `.lym/orch/role.md:324`.

### A mistake names its instruction cause; no apology (pixxelkick; Classic-Asparagus's `_observations/` folder and session reflection)

- Verdict: partly.
- Have: friction notes, `contracts/stencils/friction/friction-directive-implementer.md:14-27`, filed by the reflection agent as issues, `contracts/stencils/friction/friction-template-reflection.md:122-163`; the no-apology tone rule, `~/.claude/conversation/SKILL.md:33`; orch workarounds go into the rules file with the issue number, `.lym/orch/role.md:353`.
- Missing: the friction directive asks what went wrong, not which instruction caused it (missing, ambiguous, conflicting, or clear but not followed), and the reflection agent does not classify by that.
- Where: the four `friction-directive-*.md` files and the reflection template.
- Bench: yes, see rank 6.

### YAGNI, no unasked scope, propose scope improvements separately (pixxelkick)

- Verdict: have it.
- Where: `~/.claude/code-quality/SKILL.md:27-34`; Code-Review flags unlicensed scope, `contracts/stencils/loom/loom-rubric-code-review.md:62-63`, and so does plan review, `contracts/stencils/loom/loom-rubric-plan-review.md:82`; follow-ups go on the board as notes, `darn-template-write.md:261`.

### Fresh-eyes reviewer sub-agent on a cheap model (pixxelkick)

- Verdict: partly.
- Have: every review round is a fresh session, `contracts/recipes/loom-recipe.yaml:71`, and model choice is per role, `contracts/specs/llm-model-spec.md`.
- Missing: evidence for which reviewer model is cost-effective.
- Bench: yes, see rank 8.

### "Make no mistakes", "it's for a children's hospital", "Codex said it could do better", "use a polite manner"

- Verdict: does not fit.
- Why: jokes or motivational framing with no mechanism to test.

### Never weaken or delete a test to make it pass; tests do not get edited to pass (NamerekDev, namrog84)

NamerekDev: "it stopped editing them. it deletes them now!"; a hook added for it was deleted by the next session.

- Verdict: partly.
- Have: only the webster verify-gate fixer, `contracts/stencils/webster/webster-body-verify-fix.md:26-27`; burler's fixer corrects a wrong assertion rather than adding tests, `contracts/stencils/burler/burler-step-fix.md:44`.
- Missing: the webster implementer, the darn writer, the burler fixer and the landing merge fixer (`contracts/stencils/landing/landing-template-fixer.md:31`, "fix the failing tests only") carry no such rule, and the Code-Review rubric does not flag a weakened test.
- Where: one shared rule rendered into every editing role, the way `contracts/stencils/edit/edit-directive.md` is, plus a Code-Review rubric line; a Go diff scan in the gate would make it structural, which matters given NamerekDev's hook story.
- Bench: yes, see rank 2.

### Mutation check after writing tests (namrog84)

- Verdict: partly.
- Have: RED-before-GREEN, but only "when TDD is specified", `~/.claude/testing/SKILL.md:49-57`.
- Missing: no stencil or plan card asks for it.
- Where: the plan spec's test-card rules or the implementer stencil, or a `lym gate` step.
- Bench: yes, see rank 5.

### Review your work for creeping complexity and propose a workflow change (sammcj, partly cut off in the export); "keep it succinct" (es-ganso)

- Verdict: have it.
- Where: review rounds judge design against the rubric; friction notes carry workflow proposals; brevity is `~/.claude/prose/SKILL.md:14-35`.

### Commit messages: short subject, body or not, no Co-Authored-By, task hash in the next commit (waruyamaZero, OP, howitzer1)

- Verdict: have it.
- Where: subject shape `N: <name>`, `webster-body-implementer.md:68`; the landing writes the description and the trailer, `contracts/stencils/landing/landing-template-describe.md:49`.

### The agent you talk to never writes code; it supervises sub-agents (maniac56)

- Verdict: have it.
- Where: "You diagnose; the run's own agents change the run", `.lym/orch/role.md:281`; webster's Master never reads code, `contracts/stencils/webster/webster-template-master.md:19`.

### Simplified Technical English; ban "load-bearing", em dashes, slop words (basskittens, bigtime_porgrammer, Icy-Excitement-467)

The OP replied that banning em dashes made the model use " - " instead: "swapped one tell for another".

- Verdict: partly, and not worth a bench.
- Have: `~/.claude/prose/SKILL.md:14-35`.
- Why not more: the operator's own skills use "load-bearing" (`~/.claude/prose/SKILL.md:51`) and em dashes; a word ban trades one tell for another, as the OP saw.
  STE is the one variant with substance, since it constrains structure rather than banning tokens; it is a taste call for the operator, not a pipeline lever.

### Ask for a "go" before writing code; exploration mode; plan first, then `/grilling` (renatodinhani, holyknight00, getastatic)

- Verdict: have it.
- Where: the pipeline is this: Discussion explores before asking, `contracts/stencils/loom/loom-template-discussion.md:50-54`, then Plan, then Plan-Burler review rounds, which interrogate the plan before Webster writes code.

### "Do not guess"; prefer official docs; say when an answer rests only on training data (Whiteshovel66, TalShar)

- Verdict: partly.
- Have: "Read the index below before guessing a `lym` command", `plugins/ly/skills/cli/SKILL.md:8`; Discussion reads code before asking, `loom-template-discussion.md:52-53`.
- Missing: nothing asks an agent to flag a claim about an external library or tool that it did not verify (go-git, cobra, tree-sitter, tmux, Claude Code flags).
- Where: `~/.claude/code-quality/SKILL.md` or the implementer stencil.
- Bench: possible but weak: count first-gate compile failures on undefined symbols of external packages.

### Cite `file:line` before claiming "we do X" (Stunning-Sherbet1853)

- Verdict: partly.
- Have: review findings need `path:line` evidence, `contracts/stencils/burler/burler-fix-check-rubric.md:18`, and disputes need cited text, `burler-step-fix.md:31`.
- Missing: change descriptions, decision records, board notes and orch replies make code claims with no citation rule; Discussion deliberately avoids `file:line` (`loom-template-discussion.md:57`), so the rule must target claims of presence or absence, not signatures.
- Where: `landing-template-describe*.md`, `darn-template-write.md` Step 5, the orch role's board section, and `~/.claude/conversation/SKILL.md` for chat.
- Bench: yes, see rank 4.

### "Clearly call out the next action" (4bitfocus)

- Verdict: does not fit.
- Why: aimed at a solo chat loop where the human approves each step; runs are autonomous, and orch status reports already carry the attach command, `.lym/orch/role.md:393`.

### A repo map: one-line description per file, kept current (No_Dig_7017)

- Verdict: partly.
- Have: `lym quarry toc`, `contracts/stencils/loom/loom-template-plan.md:65`, fed by file headers and doc comments written to that bar, `~/.claude/code-quality/SKILL.md:113-120`.
- Missing: only the planner is pointed at it; the darn writer and burler reviewers are not.
- Where: `darn-template-write.md` Step 4 and `burler-step-1-explore.md`'s tool-use rules.
- Bench: possible: darn token spend and Reads before the first Edit, with and without the pointer; fold into the rank-3 bench rather than running it alone.

### No backward compatibility unless asked; update every call site (danielllenton)

- Verdict: have it.
- Where: `~/.claude/code-quality/SKILL.md:23-25`.

### Known-errors wiki per language, read before coding, appended after each own mistake (IxBetaXI)

- Verdict: does not fit.
- Why: PATTERN plus `pattern/` and the orch rules play that role, with a size bound (`internal/pattern/headroom.go:7`); an agent-appended log would grow without review, which the headroom lint exists to stop.

### Skill-run logs and a weekly cron turning repeated actions into skills (bkrsh099)

- Verdict: does not fit.
- Why: friction notes already carry the signal; the commenter says most generated skills get rejected.

### "Refactoring is harm": write `foo-2` beside `foo` (cjmurrr)

- Verdict: does not fit.
- Why: conflicts with "no `_old`/`_v2` duplication — change the call sites", `~/.claude/code-quality/SKILL.md:24`; the thread itself rejects it.

### Do not rely on CLAUDE.md for secrets; block at the filesystem or proxy (radim11)

- Verdict: have it.
- Where: guards live in code, `internal/shellguard` (package doc), and reed strips provider environment keys, `pattern/PATTERN-shuttle-provider-seam.md:9`.

### No system-wide `grep`/`find` (alanvnk: "nearly doubled the amount of sessions I can run")

- Verdict: worth adopting, if transcripts show it happens.
- Have: the plan spec scopes grep to a package, `contracts/specs/loom-plan-spec.md:370`.
- Where: a `internal/shellguard` row denying a recursive `grep`/`find`/`rg` rooted at `/` or `~`, since many parallel lym sessions share one machine.
- Bench: not needed; grep a sample of strand transcripts for such calls first.

### Replace CLAUDE.md lines with hooks and gates; a hook flags comments that will go stale (Neat-Culture9983 and parent)

- Verdict: have it.
- Where: PATTERN entries enforced by tests, `CLAUDE.md:8`; `internal/shellguard`, `internal/commentlint`, `internal/patternlint`; the perishable-specific rule, `~/.claude/prose/SKILL.md` "Don't pin a perishable specific".

### Docs and code disagree: follow the code, and report every inconsistency (SirRobinBlibs, arXiv 2610.30571: following the code went from 4.2% to 91.4% over 320 runs)

- Verdict: worth adopting, both halves (corrected by the operator, 2026-10-26).
- A description of behavior, in a `doc.go`, a `pattern/` file or `CLAUDE.md`, is not a rule: the code is the fact of what happens, so an agent that needs to know what the code does trusts the code, never a stale description.
  The code may still be wrong, so the mismatch is a finding, not a verdict: the agent silently fixes neither side and reports which side it believes is wrong.
  This is apart from `loom-rubric-code-review.md:66`, where a written rule on how new code should be written outranks a convention inferred from the surrounding code.
- Missing: no editing role is told to report a doc/code disagreement it meets, so a stale `doc.go` or `pattern/` line goes unreported.
- Where: one line in the implementer, darn writer and burler fixer stencils sending each disagreement to the change description or a friction note.
- Bench: yes, see rank 1.

### Call things what they are; do not invent terminology (actual-time-traveler, Classic-Asparagus)

- Verdict: does not fit for the project, partly for agents.
- Why: LoomYard coins its own names on purpose (weft, warp, reed, creel, dobby, batten); `CLAUDE.md:59-61` already confines the shorthands to conversation.
  An agent coining a new term in a stencil or doc comment is a review-level concern, not a bench.

### Ask one question before editing instead of guessing; say what you did not test (London-Boat)

- Verdict: have it.
- Where: the question half is `~/.claude/conversation/SKILL.md:32` for chat and Discussion for runs (darn deliberately never blocks on a dialog, `darn-template-write.md:14`); the untested half is "anything to check or run by hand that was not run", `landing-template-describe.md:42` and `landing-template-describe-darn.md:39`.

### Hook at 50% context: wind down, hand off (puts_on_rddt)

- Verdict: have it.
- Where: orch distill and refresh (`plugins/ly/skills/distill`, `plugins/ly/skills/refresh`), the orch handoff templates under `contracts/stencils/orch/`, and `scribe:handoff`; spawned strands are short-lived by design.

### "Do not suggest waiting until tomorrow"; human time estimates (extremelyhotdad)

- Verdict: does not fit.
- Why: no agent schedules its own work; dobby drives runs.

### No new markdown file without asking; one fact, one owner file (SimplifAI_Life; pixxelkick on markdown litter)

- Verdict: have it.
- Where: `~/.claude/prose/SKILL.md:33-35`, `~/.claude/code-quality/SKILL.md:66-68`, `PATTERN-stencil-ownership` in `PATTERN.md`, scratch files under `.scratch/` (`contracts/stencils/edit/edit-directive.md:8`), and friction notes at a fixed path "with no filename of your own choosing".

### Every mistake becomes a rule or, better, a pre-work check (Weekly_Cat6537)

- Verdict: have it.
- Where: `CLAUDE.md:10` (a new invariant goes into PATTERN with its code), PATTERN entries marked `(test)`, and the orch workaround rule, `.lym/orch/role.md:353`.

### Keep CLAUDE.md under a page or it gets ignored (OP)

- Verdict: partly.
- Have: `CLAUDE.md` is short and PATTERN growth is linted, `internal/pattern/headroom.go:7`.
- Missing: the orch role is 530 lines with hub rules inlined, and the stencils are long; nothing measures whether length costs compliance.
- Bench: possible and useful: a trimmed against a full orch role on replayed scenarios, counting violations of a fixed set of role rules.
  It is expensive to set up, so it ranks below the eight above.

### Read the related tests before touching code (masiha97)

- Verdict: partly.
- Have: plan cards name the tests they change or reuse, `loom-template-plan.md:176-182`; burler's pre-read covers tests, `burler-fork-reading.md:14`.
- Missing: the darn writer and the merge fixer are not told to.
- Where: `darn-template-write.md` Step 4 and `landing-template-fixer.md`.
- Bench: yes, see rank 7.

### Explicit failure path: if X fails try Y, then report and stop (Joozio)

- Verdict: have it.
- Where: `PATTERN-refusal-way-forward` in `PATTERN.md`; the bounded self-fix and `NEEDS_PARENT`, `webster-body-implementer.md:108-130`.

### Block bulk edits with a hook so each file edit is visible (ghost_operative)

- Verdict: does not fit.
- Why: LoomYard just allowed scripted edits on purpose (`contracts/stencils/edit/edit-directive.md`, commit `a1b6053a`); the scope guard and review read the diff instead.

### Some CLAUDE.md content belongs in a custom output style (thebigslapper)

- Verdict: does not fit.
- Why: spawned agents get their instructions from stencils, not from CLAUDE.md or output styles.

## 3. Cheap enough to adopt without a bench

- One shared "never delete, skip or weaken a test or assertion to make it pass" rule for every editing role, copied from `webster-body-verify-fix.md:26-27`, plus a Code-Review rubric line flagging it.
  A bench is still worth running to see whether a prose rule suffices or a diff scan is needed.
- One line in the editing stencils: report each doc/code disagreement you meet in the change description or a friction note, trusting the code for what happens and naming the side it believes is wrong.
- Point the darn writer at `lym quarry toc` and at reading the tests of the code it changes, in `darn-template-write.md` Step 4.
- A friction-note prompt line asking for the instruction cause: missing, ambiguous, conflicting, or clear but not followed.
- A `shellguard` row against recursive searches rooted at `/` or the home directory, once a transcript sample shows they happen.
