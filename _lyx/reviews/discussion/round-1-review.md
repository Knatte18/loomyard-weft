---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:82-85 (D4 category test and Bound)
    summary: D4's category test classifies by key kind and passes timeouts and seed-only content as session tuning, so loom.parent_review_wait_min (whose expiry passes the parent approval gate) and burler's lenses and fans (the review prose and lens set forks are given) stay open to the untracked local layer, falsifying D4's stated Bound.
    fix: "In D4, extend the guard category to read: switches off or hollows a guard, including a review or an approval gate, a timeout whose expiry lets that gate pass, and the prose or lens set a reviewer is given. Add loom's LocalDenied: parent_review_wait_min. Add burler's LocalDenied: lenses, fans. Replace the sentence 'the other per-worktree modules deny none' with one naming loom and burler and saying the remaining modules deny none. In D2, D5 and the acceptance criterion 'A seed-only module's local file and run_settings accept an alias, lens or fan the template lacks', narrow the seed-only exemption to models aliases. Add loom.parent_review_wait_min and burler.lenses to the acceptance criterion listing refused local keys. Update the PATTERN-config-layers line under Constraints with the extended category wording."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:97 (D5) against :136 (D8) and :226 (acceptance criteria)
    summary: D8 and D9 accept a run-settings key path that sits under an open map, such as batcher.profiles.<name> or shuttle.claude_prompt_cache_ttl_roles.<role>, but D5 and the acceptance criterion refuse any local key the template lacks with only a seed-only exemption, so Seed-Child writes a local file every later load of that module in the pair rejects with ErrInvalid.
    fix: "In D5, change 'A seed-only module's local file is exempt from the template-lacks refusal, because its keys are the operator's (D2).' to also exempt a key path under one of the module's open maps (configreg.Module.OpenMaps), matching D8's validation. In the acceptance criteria, add a row: a local file and run_settings accept an entry under an open map that the template lacks, and the module loads."
---

### [BLOCKING] D4's category test misses a gate-passing timeout and review prose

**Location:** `_lyx/discussion/decision-record.md:82-85` (D4), with D2 `:58-59`, D5 `:98` and the acceptance criterion at `:227`.

**Issue:** D4 says every key of every module was tested against its categories, and its Bound says a local override can never switch off a review or an approval gate and can change only session tuning such as timeouts, budgets, model picks and colors.
Two per-worktree keys outside every deny list contradict that Bound:

- `loom.parent_review_wait_min` bounds how long Discussion-Write's parent-review gate waits.
  `internal/parentreview/gate.go:173-180` passes the discussion with a Warn once `WaitBound` elapses ("parent review timed out; letting the discussion through"), and `internal/loomengine/config.go:678` floors the key at 1, not at a safe value.
  A local `loom.local.yaml`, written untracked by an agent in the pair or set through `run_settings`, can set it to 1 and let the discussion through the parent's approval gate after one minute.
  The key reads as a timeout, so D4's test filed it under session tuning; the test needs a category for a timeout whose expiry passes a gate.
- `burler` is seed-only, denies nothing under D4, and D5 exempts its local file from the template-lacks refusal.
  `burlerengine.Config.Lenses` holds the review prose each fork is given and `Fans` the lens set (`internal/burlerengine/config.go:39-52`).
  A local override can replace a lens's prose with vacuous text or drop lenses from a fan, hollowing a review untracked and unseen at landing, where today the tracked `burler.yaml` shows any such change.

Both instances come from one method gap: the categories classify a key by its kind (executable, file, timeout, model), not by whether its value can pass or hollow a gate.
The rubric's attack-surface question requires the Bound to hold; as written it is false for these keys, the same class of defect the operator's appended decision closed for `require_pr_to_base` and `darn.review_max_bounces`.
I checked the other per-worktree keys that look like guards: `loom.review_max_bounces`, `loom.review_circling_checkpoint`, `webster.verify_gate_attempts` and `self_fix_cap` are all refused below 1 at load (`internal/loomengine/config.go:675-693`, `internal/websterengine/config.go:87-99`), and `reed.operator_quiet_s` falls back to 5 at or below 0 (`internal/reedengine/io.go:244`), so none of them can switch a gate off.

### [MEDIUM] D5 refuses the open-map entries D8 and D9 accept

**Location:** `_lyx/discussion/decision-record.md:97-98` (D5), `:136` (D8), `:145` (D9) and `:226` (acceptance criteria).

**Issue:** D8 validates a run-settings key path as valid when it "exists in its template, sits under one of its open maps or belongs to a seed-only module", and D9 validates as D8 does before writing the local file.
D5 refuses at load any local key "its module's template lacks", exempting only a seed-only module, and the acceptance criterion repeats that refusal.
The registered open maps include `batcher.profiles` and `shuttle.claude_prompt_cache_ttl_roles` (`ConfigOpenMaps` in `internal/batcher/config.go` and `internal/shuttleengine/config.go`), whose entries are open-ended by design.
So `run_settings` `{"shuttle": {"claude_prompt_cache_ttl_roles": {"plan": "1h"}}}` passes the board write and Seed-Child, and then every shuttle load in the pair fails with `ErrInvalid`.
The plan writer gets two contradictory rules and no way to pick between them from the record.

## Rejected

- Focus 2, D9's removal of unnamed local files: D9 names what it discards (a hand-written override placed while Seed-Child was Stuck), and the bound beyond the Warn holds: Worktree-Create's pre-existing-branch refusal (`internal/battenshed/create.go:49`) and the batten recipe order (`contracts/recipes/batten-recipe.yaml`, Worktree-Create then Seed-Child then Run-Shed) leave only that window. Not a defect.
- Focus 3, D6's strip: D6 states that an equal pin becomes inherited and that seeding adds `inherited` keys, and its marker bound covers a pair forked before, after or during the walk (a pair skipped as being created is stripped by the next walk). Not a defect.
- Focus 5, premises: Seed-Child reads the task's recipe fresh on every call (`SeedChildDeps.ReadBoardType`, `internal/battenshed/deps.go`) and precedes Run-Shed, the only row that launches a child driver; `internal/configreg` imports `internal/boardengine` (`go list`), so `boardengine` cannot import `configreg`. Both premises hold.
- Focus 4, D1, D2, D5 and D7 for a hub-wide module: the base layer is the hub file, `--layer default` writes it, `--layer worktree` is refused, and the acceptance criteria match; the only disagreement found is F2.
- `shuttle.startup_timeout_s: 0` zeroes the orphan sweep's age guard, but the sweep is worktree-local and the same value fast-fails every startup, so it harms only the pair setting it and widens no trust boundary.
- The four "Added after Discussion" sections restate content already folded into D4 and D6; they agree with it and are left alone, since they record the parent's and operator's rulings.
- Focus 6, support log versus record: every requirement in the lens-review dispositions (marker, Seed-Child only, notice, local-only unknown-key refusal, `lyx-cleanup`, injected closures, refusal-spec row, `agentname` guard, Warn per removed file) is present in the record's Decisions or Constraints; no relocation finding is raised.
