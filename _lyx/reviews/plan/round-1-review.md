---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/04-hibernate-engine-watch.md:66
    summary: When the Added callback's own request write fails, Pause returns an error with its hub entry live, but Watch only lifts the entry recorded as Owned, so the entry is left live with no owner and the failure notice says no pause was owned.
    fix: In card 4's Firing paragraph, have the Added callback keep the EntryRef it builds in a local variable before it calls updateRequest, and have the Pause-error path lift that EntryRef through liftOwned (or Lift) when the request records no Owned entry, so the failure notice reads its pause was lifted. Card 1 already says the caller holds the entry from the argument for this case. In the TestWatch Pause-error row, add a sub-row where a fakeWorld hook that runs in the fake Pause before it calls req.Added replaces the request file with a directory of the same name, so the Added write fails. Assert that Watch returns an error and that the hub entry is gone from the pause record's live list.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/04-hibernate-engine-watch.md:114
    summary: The after-all guard branch that skips a listed slug with a missing status has no named input or test, and no TestWatch row reaches it.
    fix: Extend card 4's --after-all TestWatch row (line 114) with a sub-row where RunSlugs lists a slug whose status the fakeWorld missing-slug switch reports as not found, beside one run reading done. Assert that the trigger fires on the first reading and that the request does not end failed.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/07-shortcuts-p-r.md:10
    summary: Card 7 adds two new root commands, lyx p and lyx r, but its Intent never says what they can let through or what bounds them, which the attack-surface rule requires of every new verb.
    fix: Add one sentence to card 7's Intent. It says that each shortcut runs lyx tender pause or lyx tender resume in the caller's own process environment, so tender's caller check still applies unchanged (an agent may pause or resume only its own pair) and the shortcuts open no route past it.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/03-hibernate-engine-verbs.md:55
    summary: Card 3, an engine card, lists the unexported cli symbol internal/tendercli#callerOf under Uses, although its Intent mirrors that rule in refuseCaller and an engine never imports a cli package (PATTERN-cli-cobra).
    fix: Delete the Uses bullet internal/tendercli#callerOf from card 3. The Intent's prose reference to tendercli's rule stays. validate-plan --overlay with this change reports no blocking finding.
  - id: F5
    severity: NIT
    class: scope
    location: _lyx/plan/07-shortcuts-p-r.md:15
    summary: Card 7 updates two shortcut lists in docs/overview.md, but the tender module bullet also names the shortcuts (lyx pause and lyx resume are shortcuts for the first two) and gets no mention of lyx p and lyx r.
    fix: In card 7's Intent sentence on docs/overview.md, add the tender module bullet to the lists that change, so it reads that lyx pause and lyx p, and lyx resume and lyx r, are shortcuts for the first two verbs. docs/overview.md is already an Edit target, so no target changes.
---

### [LOW] Added-callback write failure leaves the pause entry live and unowned

**Location:** `_lyx/plan/04-hibernate-engine-watch.md:66`

**Issue:** Card 1 specifies that an `Added` error makes `Pause` return at once with the entry still live, "and the caller holds it from the argument it was given".
Card 4's `Added` closure writes `Owned` through `updateRequest`.
If that write fails, `Pause` returns the wrapped error, and card 4's Pause-error path "lifts the owned entry", which the request never recorded.
The hub entry the daemon created then stays live with no owner.
The failure notice takes its "it owned no pause to lift" form, and the hub stays paused until the operator runs `lyx resume`.
Decisions 6 and 7 have the daemon lift the entry its pause added on every failure path.
Card 1 built the argument hand-off for exactly this case, but card 4 never uses it.

### [LOW] after-all skip of a missing status is unreached

**Location:** `_lyx/plan/04-hibernate-engine-watch.md:114`

**Issue:** Line 56 prescribes a guard branch: "a listed slug whose status is missing is skipped".
This is the opposite of the `--after <slug>` case, which ends `failed`.
None of the `TestWatch` rows lists a slug under `--after-all` whose status `RunStatus` reports missing, so nothing pins the skip.
An implementation that copied the `--after` branch and failed the request would still pass.
The missing-slug switch that `fakeWorld` (card 3) already declares makes a row for this input easy to add.

### [LOW] lyx p and lyx r carry no stated bound

**Location:** `_lyx/plan/07-shortcuts-p-r.md:10`

**Issue:** The rubric's attack-surface question applies to every new verb.
Card 7 mounts two new root commands but only says the names cannot shadow a module.
It never says what the commands can let through, or that tender's own caller check (`refuseCaller` in `internal/tendercli/cli.go`) still applies.
In the code, that check runs inside `lyx tender pause`/`resume` with the same strand variable, so the bound holds, but the card has to state it.

### [NIT] Engine card lists a cli symbol under Uses

**Location:** `_lyx/plan/03-hibernate-engine-verbs.md:55`

**Issue:** `callerOf` is unexported in `internal/tendercli`, so `hibernateengine` cannot call it.
PATTERN-cli-cobra says an engine never imports a cli package.
The Intent restates the rule in `refuseCaller`, so this Uses entry only adds a misleading dependency edge into a cli package.
I confirmed with `validate-plan --overlay` that the fix adds no blocking finding.

### [NIT] Tender bullet in docs/overview.md misses the new shortcuts

**Location:** `_lyx/plan/07-shortcuts-p-r.md:15`

**Issue:** The tender bullet in `docs/overview.md` (line 405) ends with "`lyx pause` and `lyx resume` are shortcuts for the first two".
After card 7, that statement leaves out `lyx p` and `lyx r`.
Card 7 names only the source-tree line and the shortcuts bullet.

## Focus verdicts

- Fidelity: every Decision 1–13 and every Constraint is carried by a card.
  Cards 1, 3, 4 and 5 carry the module, verbs, callers, request file, daemon, firing, lift and cancel.
  Card 6 carries the deny, card 8 and card 5's audience annotations carry the orch, card 7 the shortcuts, and card 5 the sandbox exclusion.
  Two edits sit outside Scope: the `refusal-spec.md` section and the `deniedRecoveryForms` addition.
  Both are forced by the repo's own contracts: the existing `## pause` precedent, and the list's stated rule that it follows claudeengine's deny table.
  Neither is unlicensed scope.
- Attack surface: card 5's hidden `watch` verb, together with card 4's Ownership paragraph, states that `watch` acts only for its own non-terminal request.
  Card 6 states the deny's guardrail bound and what it lets through.
  Card 1 bounds the `Added` hook, and card 5 bounds the land-lock probe.
  The one gap is card 7 (F3).
- Card order: card 3 consumes card 1's `Lift`/`EntryRef`/`PauseRequest`, and card 5 consumes card 2's `HubDeps` and cards 3–4.
  No card depends on a later one.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` cover every package the cards target, including both new packages, tender, shortcutcli and claudeengine.
  The tmux and llm tiers are vetted, and the section follows the template's order.
- Test economy and reachability: every refusal in cards 3 and 4 has a named input and row, apart from the after-all missing-status skip (F2) and the Added-write-failure path (F1).
  No new test duplicates an existing one.
  Card 1 folds the lift rows into `TestResume` and `TestPause`, and the budget rows (`claudeengine` 111→112, `hibernateengine` 3→4, `hibernatecli` 1) match the counts on disk.
- Edit glyphs: cards 1, 6, 7 and 8 list member glyphs for every named symbol and test they change.
  No signature changes, so no re-sign arrow is due: card 1 adds a field to a struct and moves `Resume`'s body without changing its signature.
  Card 2 is a `Rename`, not an `Edit`, so it needs no ImpactSummary.
  Each ImpactSummary states a real blast-radius conclusion.
