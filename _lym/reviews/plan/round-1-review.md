---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/08-sandbox-reaper.md:50
    summary: "Card 8's fenced `PATTERN.md` line for `PATTERN-child-processes` rewrites more than the declared change: besides adding the `EndProcess` orphans clause, it turns \"a pinned or spawned lym command names its binary via `selfexec.LymBinary`\" into \"a spawned lym is named via `selfexec.LymBinary`\", dropping pinned commands from the rule summary, a narrowing no declared PATTERN change licenses."
    fix: "In card 8's fenced block, replace the prescribed line with this one, whose rule text is exactly 280 characters and so passes `pattern-entry-line-cap`: - `PATTERN-child-processes` — Starting or ending a process: log the spawn via `internal/logger`; its starter waits on it, bar a double fork's intermediate; a run's processes register in `internal/runproc` and end only via it, orphans via `EndProcess`; a pinned or spawned lym is named via `selfexec.LymBinary`. — [background](pattern/PATTERN-child-processes.md)"
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/07-sandbox-builder.md:27
    summary: "Card 7 prescribes `SandboxDeps.Clone func(warp, weft, container string)` in `internal/hardenshed`, which is outside the fabric owner set; PATTERN-fabric-boundary (Fabric vocabulary) bars side-named spelling in a non-owner file, and `TestEnforcement_FabricVocabulary` scans non-comment code."
    fix: "In card 7's Intent, rename `Clone`'s parameters by role, for example `Clone func(codeBare, recordsBare, container string) (hubPath string, err error)`, and say the bares by role in `hardenshed`'s doc and code. `fabricengine.SeedLocalBares` may keep its signature, since fabricengine is in the owner set. In card 16's `new` wiring, keep the side names out of `internal/hardencli` too."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/16-harden-cli.md:22
    summary: "PATTERN-cli-cobra's Package naming rule says \"`<module>cli` imports `<module>engine`\" and lists each deviation. `internal/hardencli` uses `internal/hardenshed` and `internal/loomcli` and has no `hardenengine`, and no card adds its deviation line to `pattern/PATTERN-cli-cobra.md`."
    fix: "Add `pattern/PATTERN-cli-cobra.md` to card 16's **Edit:** list. Add a deviation bullet to the Package naming list: \"`hardencli` uses `internal/hardenshed` and `internal/loomcli`, with no engine package of its own.\" Say this in card 16's Intent. If the parent rules that the bullet is a PATTERN change the record must declare, ask the parent with `lym creel post --parent --expect-reply` to record it with `lym loom decision add`, then copy it into the overview's `## PATTERN changes`."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md:60
    summary: "D3 and D4 decide that lym's own hub sets `harden-burler-fix` to `[0, 1]` and `harden-live` and `harden-live-copies` to `[2]`. Shared Decision `hub-gate-keys-are-a-hub-write` drops these steps, and the record has no `Added after Discussion` entry for the drop. Preflight's hub-key refusal covers only the two live keys, so `harden-burler-fix` silently stays at the template's `[0]` and the verify gate never runs tier 1, against D3."
    fix: "Ask the parent with `lym creel post --parent --expect-reply`: the hub's `_bolt/_lym/config/gate.yaml` lies outside the task worktree, so no card can write the three values. The parent then either makes the three `lym config gate --set` writes from the prime, or records with `lym loom decision add` that they are operator writes outside the plan. Once the parent has recorded the call, cite its added entry's title in the Shared Decision `hub-gate-keys-are-a-hub-write`."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-harden-rows.md:55
    summary: "`Campaign.LiveRed` reads an absent live report as not red. When the fix row's `verify` gate exhausts, `BurlerFix` returns a counted Stuck before the `live` gate runs (`internal/shedadapters/burlerfix.go:326-332`), so the round writes no live report. With `auto_accept` set and the budget spent, the Harden-Bouncer then auto-accepts a round Go never verified live, against D6 (\"no campaign lands past a red live gate without a parent decision\") and D4 (a run that verified nothing is never read as green)."
    fix: "In card 11, make `LiveRed` answer true for a round whose live report is absent once that round's fix row ran: the round's fixer report or gate-findings file exists and `harden-live-<round>.json` does not. An absent report with no fix row run stays not red. Add a row for this case to `TestCampaignRoundFacts`, or to a `LiveRed` row there, and a matching red row under `TestBouncer_AutoAccept` in card 10."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-live-gate.md:33
    summary: "Card 6 returns \"a gate-config refusal of the call\" as a wrapped `Run` error and writes no report. But card 14 wires `Run` to spawn `lym gate test` and return only whether it passed and its output, so a refused call reads as a failed run. A refused `harden-live` call has no started line, so the report goes red and the fixer is re-prompted over a hub config fault it cannot fix; a refused copies call is recorded as a copies failure."
    fix: "In card 6, give `LiveRun` a `Refused` field holding the refusal text, which card 14's `Run` fills from the spawned call's JSON envelope (`ok: false` with an `error` and no tier run). The gate returns a refused `harden-live` call as the wrapped error with its stated way forward. It records a refused copies call in the live report as refused rather than failed. Add a row for each to `TestLiveGate`, and one to `TestHardenLiveArgs` or `TestWire_HardenerRecipe` in card 14 asserting that the envelope is parsed."
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/02-gate-harden-keys.md:14
    summary: "Card 2's three new `diff`-scope template keys falsify `internal/gateslot/doc.go:109`, which names the four keys the template gives `diff`, and `:119`, which lists `mutation` last on \"every key of scope `diff`\" and names those four keys. An `Extends` cannot rewrite those lines. The doc's site table and `pattern/PATTERN-test-speed.md`'s key-and-site table (lines 48-58) gain no rows for the live gate and Harden-Burler-Fix sites, and the card does not say which `reports` the new keys carry."
    fix: "On card 2, move `internal/gateslot/doc.go` from **Extends:** to **Edit:**, and add `pattern/PATTERN-test-speed.md` under **Edit:**. State in the Intent which `reports` the three keys carry, and rewrite the two `doc.go` sentences to match. Add rows for `harden-live` and `harden-live-copies` (the Harden-Burler-Fix row's `live` gate, `diff`) and `harden-burler-fix` (its `verify` gate, `diff`) to both tables."
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-hardener-config.md:1
    summary: "Cards 3 and 16 create the first tests of the new packages `internal/hardenshed` and `internal/hardencli`, but neither creates a `TestMain` that calls `tmuxkit.Main`. PATTERN-test-isolation requires one in every test package, and `cmd/lym/tmuxisolation_test.go` enforces it."
    fix: "Add `plan:internal/hardenshed#TestMain` -> `func TestMain(m *testing.M)` to card 3's **Create:**, and `plan:internal/hardencli#TestMain` -> `func TestMain(m *testing.M)` to card 16's **Create:**. Each lives in its package's `testmain_test.go` and calls `tmuxkit.Main`, as every other package's does."
  - id: F9
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/05-campaign-preflight.md:50
    summary: "Card 1 refuses `--copies` over a key that names more than one tier. Preflight refuses the copies key only when it is unset or names tier 3, so a hub whose `harden-live-copies` names `[1, 2]` passes Preflight and has its copies call refused in every round, with no up-front way forward."
    fix: "In card 5's refusal 3, also refuse, when copies run, a `harden-live-copies` key that names more than one tier, with the same `lym config gate --set` way forward. D4 already makes such a key unusable for `--copies`. Add a row to `TestCheckCampaign`. If the fixer judges this outside D2's list, ask the parent to record it instead."
  - id: F10
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-sandbox-reaper.md:42
    summary: "`Reap` step 2 spares the caller for `runproc.End`, but step 4's `EndProcess` kill of unrecorded root-naming processes names no spare. PATTERN-child-processes has every ending verb spare the caller and its ancestors."
    fix: "In card 8, step 4 skips the caller's pid and its ancestors, which it reads from the same process listing. Add a `TestReap` row in which an unrecorded process naming the root is the caller's ancestor and is left alone."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/19-treadle-delete.md:53
    summary: "Card 19 adds a way forward only to `skillLists`' refusal of a `skills` key, which is CategoryGuard and so is never reached from a local layer. An `efforts` key is CategoryNone, so a task's `run_settings` can put `treadle-judge` into the local layer. The reinterpretation leaves that layer alone, and `checkEfforts`' refusal still names no way forward."
    fix: "Add `internal/agentconfig#checkEfforts` to card 19's **Edit:** list. Its refusal of an `efforts` key naming a role lym does not start gains the same way forward: remove the key from that layer's agents.yaml with lym config agents. Add a row to `TestLoad_RefusesUnusableContent` asserting it."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/19-treadle-delete.md:59
    summary: "Card 19 misses one comment that names treadle: the doc comment of `mergeresolveAllowedImports` in `internal/mergeresolve/seam_enforcement_test.go:26` cites \"the same call treadleengine's seam allowlist already makes\", which names a package card 19 deletes."
    fix: "Add `internal/mergeresolve#mergeresolveAllowedImports` to card 19's **Edit:** list; `lym quarry resolve` puts the comment inside its span, lines 20-38. Add it to the Intent's list of comments reworded to stand alone."
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/08-sandbox-reaper.md:21
    summary: "Once the Hardener reaper also uses `EndProcess`, `internal/runproc/doc.go:64` is false: it says `EndProcess` runs \"for the orphans `lym tender reap` ends\". Card 8 updates only `PATTERN.md` and the background file."
    fix: "Add `internal/runproc/doc.go` to card 8's **Extends:** list, and add to the Intent that its `EndProcess` sentence names the Hardener sandbox reaper's orphans beside `lym tender reap`'s."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/03-hardener-config.md:42
    summary: "`hardenshed.LoadConfig` loads strictly, but PATTERN-config-layers' Strictness section lists the strict callers and no card adds `hardenshed` to that list."
    fix: "Add `pattern/PATTERN-config-layers.md` to card 3's **Edit:** list, and add `hardenshed` to the Strictness section's strict callers."
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/13-hardener-recipe.md:64
    summary: "D1 puts the hardener recipe's board value in scope, and the board entry's Takes names `boardengine/task.go`. But two board texts that list task recipes are left without `hardener`: `normalizeTask`'s refusal (`internal/boardengine/task.go:351`, \"a task's recipe is tapestry, motif, darn or sampler\") and the `recipe` field help in `storeVerbs` (`internal/boardcli/cli.go:604`)."
    fix: "Add `internal/boardengine#normalizeTask` and `internal/boardcli#storeVerbs` to card 13's **Edit:** list, so both texts name `hardener` beside the other task recipes. Extend whichever test asserts `normalizeTask`'s refusal text."
---

### [BLOCKING] Card 8's PATTERN line narrows the rule beyond the declared change

**Location:** `_lym/plan/08-sandbox-reaper.md:50`

**Issue:** The declared change only admits the Hardener sandbox reaper beside `lym tender reap` as a user of `EndProcess`. The prescribed overview line also drops "pinned" from the `selfexec.LymBinary` clause (`PATTERN.md:52`), which narrows the rule. The current text is 274 characters, and the shortening was likely made to fit the 280-character cap (`internal/pattern/check.go:25`). A 280-character line keeps "pinned" and adds the orphans clause.

### [BLOCKING] Fabric side tokens in hardenshed's sandbox seams

**Location:** `_lym/plan/07-sandbox-builder.md:27`

**Issue:** `SandboxDeps.Clone` spells both side names as parameters inside `internal/hardenshed`, which is outside PATTERN-fabric-boundary's owner set. The vocabulary scan reads non-comment code (`internal/lymcwd/enforcement_test.go` strips only comments), and no current non-owner package in `loomcli`, `shedrecipe` or `burlercli` spells the names in code.

### [BLOCKING] hardencli's package-naming deviation is not listed

**Location:** `_lym/plan/16-harden-cli.md:22`

**Issue:** PATTERN-cli-cobra lists each `<module>cli` that does not import a `<module>engine`, for example `battencli` using `battenshed`. `hardencli` pairs with `hardenshed` and calls `loomcli.HardenReapDeps`, and no card adds its deviation line.

### [MEDIUM] The hub gate-key values D3 and D4 decide are dropped with no recorded decision

**Location:** `_lym/plan/00-overview.md:60`

**Issue:** The Shared Decision is right that the hub's `gate.yaml` lies under `_bolt/_lym/config/`, outside the task worktree. But the rubric accepts an uncarried Decision step only through an `Added after Discussion` entry about the task's own board entry. Neither applies here. The `harden-burler-fix` value also has no Preflight check, so the shortfall is silent.

### [MEDIUM] A round whose verify gate exhausted reads as live-green

**Location:** `_lym/plan/11-harden-rows.md:55`

**Issue:** The `live` gate comes after `verify` in the fixed gate order, and `BurlerFix` turns an exhausted gate into a counted Stuck before later gates run. The Bouncer's auto-accept sites, `settleUndecided` and `softCapRule`, then consult a `LiveRed` that answers false for the missing report.

### [MEDIUM] A refused live call is indistinguishable from a failing test run

**Location:** `_lym/plan/06-live-gate.md:33`

**Issue:** Card 6's error branch for a gate-config refusal cannot be reached through card 14's `Run` wiring, which reports only a pass flag and output. A refused `harden-live` call therefore spends the fixer's `live_attempts` on a fault only the prime can fix.

### [MEDIUM] Gate docs and the test-speed key table go stale with the new keys

**Location:** `_lym/plan/02-gate-harden-keys.md:14`

**Issue:** `internal/gateslot/doc.go:109` and `:119` enumerate the template's `diff` keys and the keys that carry `mutation`. `pattern/PATTERN-test-speed.md:48-58` tables every gate key with its site. Card 2 updates none of these.

### [MEDIUM] New test packages lack the tmuxkit TestMain

**Location:** `_lym/plan/03-hardener-config.md:1`

**Issue:** Cards 3 and 16 create the packages' first tests without the `TestMain` that PATTERN-test-isolation requires and `cmd/lym/tmuxisolation_test.go` checks.

### [LOW] Preflight lets a multi-tier copies key through

**Location:** `_lym/plan/05-campaign-preflight.md:50`

**Issue:** Card 1 refuses `--copies` over a key that names several tiers, but Preflight does not, so every round's copies call is refused.

### [LOW] Reap's orphan kill states no caller spare

**Location:** `_lym/plan/08-sandbox-reaper.md:42`

**Issue:** Step 4 ends unrecorded processes that name the root through `EndProcess`, and the card spares neither the caller nor its ancestors there.

### [LOW] The efforts refusal gets no way forward

**Location:** `_lym/plan/19-treadle-delete.md:53`

**Issue:** The layer the card's added way forward is meant for can hold `efforts` keys but not `skills` keys, and the card adds the way forward to the `skills` refusal only.

### [LOW] A treadle mention in mergeresolve is missed

**Location:** `_lym/plan/19-treadle-delete.md:59`

**Issue:** `internal/mergeresolve/seam_enforcement_test.go:26` names `treadleengine` in the doc comment of `mergeresolveAllowedImports`.

### [LOW] runproc's package doc names one EndProcess user

**Location:** `_lym/plan/08-sandbox-reaper.md:21`

**Issue:** `internal/runproc/doc.go:64` ties `EndProcess` to `lym tender reap` alone.

### [LOW] The strict-caller list omits hardenshed

**Location:** `_lym/plan/03-hardener-config.md:42`

**Issue:** PATTERN-config-layers' Strictness section names the strict loaders, and `hardenshed` becomes one without a card listing it.

### [LOW] Board recipe enumerations omit hardener

**Location:** `_lym/plan/13-hardener-recipe.md:64`

**Issue:** `normalizeTask`'s refusal text and the board CLI's `recipe` help both list task recipes, and neither gains `hardener`.

## Overlay check of the proposed target changes

Run from the records anchor, `lym loom validate-plan --overlay` resolves code paths against the records side.
The `{}` baseline therefore already reports `path-missing` and `glyph-not-found` for the plan's existing code targets, such as `internal/runproc#End`, which exists in the code worktree.
The combined overlay of fixes F3, F7, F8, F11, F12, F13, F14 and F15 added findings of only those two classes, each on a path or glyph confirmed in the code worktree:
`lym quarry resolve` found `internal/boardengine#normalizeTask` (lines 304-354), `internal/boardcli#storeVerbs` (579-1167) and `internal/mergeresolve#mergeresolveAllowedImports` (20-38).
It added nothing else beyond the informational `create-new-unit` for the two `TestMain` handles.
The fixer should re-run `lym loom validate-plan` where the code paths resolve before applying these target changes.

## Rejected

- guard-reach finding 4 (tier 3 is checked only at Preflight): D4 and D2 make Preflight's refusal the decided bound on tier 3, and a hub `gate.yaml` changed to name tier 3 mid-run is an operator write outside the run, not a state the plan must guard.
