## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — scope and shape

- Q: Is a dynamic per-hub template the right thing, or should picking a language write the layer's lenses into the hub file (a one-time seed)?
  Recommended: a layer resolved at load, since the operator decision calls it a layer and a frozen seed stops later layer edits from reaching the hub.
  Picked: layer resolved at load.
- Q: Where does the pick live: `lenses.yaml` or the hub-wide `language.yaml`?
  Recommended: `lenses.yaml`, which keeps `internal/lenses` a leaf that reads no other module.
  Picked: `lenses.yaml`, key `language`.
- Q: Which languages ship? Recommended: `go`, `csharp`, `python`, the three the operator decision names. Picked as recommended.

### Turn 2 — mechanism

- Q: Compose the template in lenses (two loads) or in configengine?
  Exploration showed configsync's reconcile, prune and strip paths judge files against `m.Template()` directly, so a composition only lenses knew would let reconcile drop entries against the wrong template.
  Recommended: an overlay on `configengine.Spec`, one composing function, used by configengine and configsync's judging paths.
  Picked as recommended.
- Q: Which layers may hold `language`?
  A selector in the worktree layer over a hub file would let reconcile of the hub file judge against a template other than the one the load uses.
  Recommended: base layer only, refused above it. Picked as recommended.

### Turn 3 — content

- Q: Are the rules lenses (`pattern-fit`, `code-pattern-fit`) language lenses?
  Recommended: no; where rules live is a hub property, so the base reads PATTERN when present, else `CLAUDE.md`, and a hub overrides by name.
  Picked as recommended.
- Q: Which lenses go into the layers? Recommended: the five whose base prose names Go constructs. Picked as recommended.
- Q: Where does `numerics` live? Recommended: the `csharp` layer, added to `review-code` and `full`, removable with `remove`. Picked as recommended.

### Turn 4 — removal and edge cases

- Q: What does `remove` act on? Recommended: lenses only, stripped from every fan; an emptied fan is dropped and reported when used. Picked as recommended.
- Q: A `remove` naming no lens? Recommended: refuse, so typos and language switches surface. Picked as recommended.
- Q: Who sets `language` for this hub? Recommended: the operator via `lyx config lenses --set`, since the task cannot write Bolt's config and lyx detects no language. Picked as recommended.

Lens review (seven forks, all returned):

- interactions BLOCKING, D3×D6 way forward writes the worktree layer D3 refuses — fixed: way forward is `--layer default --set` from the prime (merged with walkthrough and bound-breaker BLOCKINGs).
- interactions MEDIUM, worktree file judged with the wrong selector — fixed: D4 reads the selector from the base layer for every judged document (merged with existing-paths BLOCKING on `BelowLayer` and walkthrough BLOCKING).
- interactions MEDIUM, Open risks contradicts D5 on `--set` refusing an emptied fan — fixed: risk now says the failure surfaces at loom/darn fan key validation (merged with bound-breaker BLOCKING).
- interactions LOW, a layer switch rolled back by a stale `remove` — fixed by the Warn-and-skip change below.
- existing-paths MEDIUM, `sparseSeed` on the wrong side — fixed: D4 classifies paths by what they do; every strip, seed included, uses the effective template.
- existing-paths LOW, `remove` invisible to raw resolved reads — fixed: D5 says only `lenses.Load` applies it.
- bound-breaker MEDIUM, refusing an unknown `remove` entry turns a later template edit into load failures — fixed: skipped and logged at Warn.
- bound-breaker MEDIUM, "base template holds every overlay key" false — fixed: D4 now says overlay names live in per-name maps that listing paths keep.
- bound-breaker LOW, `formerTemplateValues` for the reworded lenses — fixed: D1 says none get one.
- bound-breaker LOW, worktree `remove` replaces the hub's — fixed: stated in D5's bound.
- walkthrough LOW, Go hubs' rules lenses change too — fixed: stated in D6.
- goal-scope MEDIUM, Go-wording loss deferred to the orch — fixed: D6 states the loss as accepted, with why.
- goal-scope MEDIUM, single-composer rule missing from PATTERN — fixed: `PATTERN-template-overlay` added.
- goal-scope LOW, rules lenses depart from the board's candidates — fixed: added to Auto-mode assumptions.
- goal-scope LOW, numerics default cost — fixed: cost stated in Auto-mode assumptions.
- goal-scope LOW, stale header in existing hub files — fixed: Scope Out line.
- board-fit MEDIUM, csharp lenses invite guards NORCE forbids (`dotnet-verify-protocol`) — fixed: D2 prose defers to the repository's rules.
- board-fit MEDIUM, `language` key may clash with a later `language.yaml` hub-language key — fixed: key renamed `lens_set`.
- board-fit LOW, `review-setup-audit` interaction — rejected: the superset test already catches a base fan change the audit makes.
- board-fit LOW, overlap with `dotnet-verify-protocol`'s review checklist — rejected: that checklist is a rubric concern, not a lens; nothing here builds it.
- leanness LOW (several) — fixed: dropped path enumerations from D4, the duplicate Notes bullet, duplicated Constraints, restating Auto-mode bullets and the Scope mechanism bullet; kept D2's topic lists, which tell the implementer what each layer covers.

## Rejected alternatives

- Seeding the layer's lenses into the hub file on pick: freezes the prose, so later layer edits never reach the hub, and contradicts the sparse base.
- Composing the effective template inside `internal/lenses` with two loads: configsync's reconcile would still judge against the base template and could change loaded values.
- Putting the pick in `language.yaml`: makes `internal/lenses` read another module and breaks its leaf import set.
- Detecting the language from `go.mod`, `*.csproj` or `pyproject.toml`: language logic in code, which the board rules out.
- Rules lenses per language: rule location is a hub property, not a language one.
- `numerics` in the base template outside every fan: a C# hub would have to override `review-code` whole to run it, freezing that fan against base edits.
- `remove` that also removes fans: no request for it; a fan is already turned off at the fan key.
- Naming the key `language` or `set`: `set` reads ambiguously with `--set`, and `language` competes with a hub-language key a later entry may add.
- Refusing a `remove` entry that names no lens: a later template rename would fail every load of every hub that removed the name.
- Reading the selector from the file being judged: a worktree file never holds it, so it would be judged against the base while the load uses the overlay.

## Review rounds

_No rounds yet._

## Question ledger

- Layer resolved at load vs one-time seed — auto-pick: layer at load (operator decision says layer; seeds go stale).
- Key location and name — auto-pick: `lens_set` in `lenses.yaml` (leaf package; renamed from `language` after the board-fit lens).
- Languages shipped — auto-pick: `go`, `csharp`, `python` (named in the operator decision).
- Composition site — auto-pick: configengine overlay used by configsync too (reconcile correctness).
- Selector layers — auto-pick: base layer only (reconcile judges one file).
- Rules lenses — auto-pick: base, neutral wording (hub property).
- Overlaid lenses — auto-pick: `error-handling`, `test-gaps`, `performance`, `api-design`, `concurrency`.
- `numerics` placement — auto-pick: `csharp` layer, in `review-code` and `full`.
- `remove` semantics — auto-pick: lenses only; emptied fans dropped and reported on use; unknown names skipped with a Warn.
- Setting this hub's layer — auto-pick: operator runs `lyx config lenses --layer default --set lens_set=go` from the prime after landing; the interim neutral base is accepted.
