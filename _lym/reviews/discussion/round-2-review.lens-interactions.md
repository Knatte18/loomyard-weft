```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I reviewed pairs of the record's decisions that act on the same state and checked each criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 / D2 (jacquard's rows, and tapestry as Motif + Jacquard): held.
- D2 / D3 (Jacquard as the last block row with no `on_done`, gate `always`): held. D3 admits exactly that row.
- D2 / D4, loom contract: held. Tapestry's on_done path is Motif → Discussion-Write → Discussion-Bouncer → (block exit) Jacquard → Preflight → Record-Seed, so the seed row follows the record row.
- D2 / D4, Record-Seed after Discussion: held. It writes only missing files, so it does nothing in a tapestry run.
- D2 / D4, in-flight run parked at Preflight, Stuck at Record-Seed, then goto Discussion-Write: held. The Discussion exit leads back through Jacquard to Record-Seed, where the record now exists.
- D2 / D4, `undiscussed` gate seeding, then Record-Seed: held. Record-Seed finds the seeded record and writes nothing.
- D4 / D5, predicate (seed row and no record row) against `RecordEngine` returning RecordSeed only for a seed-only recipe: held. Both read the rows the same way.
- D4 decision verbs / D1 rows: held. `recordWritePending`'s literal `NamePreflight` is a row jacquard holds, and Record-Seed takes Record-Write's place through `EngineRow`.
- D4 decision verbs / D2, tapestry: held. `RecordEngine` gives DiscussionWrite, so tapestry stays on the Discussion path. No literal use of Preflight sits on that path (the grep found it only in `decision.go:605`, timingspans and interruptpolicy).
- D4 Record-Seed Stuck reason / D5 `lym loom start` way forward (jacquard outside a batten pair): **broken** (finding 1).
- D5 batten site / D5 `lym loom start` site / D5 done-motif clear: held for the refusal order. The clear sets found=false, so the stored-record check runs. The leftover task branch belongs to another lens; see the cross-lens note.
- D5 `lym loom start` check / bootstrap seed write: held. The seed is written before the check and left uncommitted, and the non-batten way forward removes it first.
- D6 / D7 (filed jacquard parts against profiles' jacquard blocks): held for shipped profiles and operator-added ones that hold a tapestry block.
- D7 / D1 (jacquard blocks carry `tapestry.*` keys, which jacquard rows read): held.
- D7 / sharedbootstrap `writeRunUsage` (`profiles.Default(jacquard)`): held, since `defaults.jacquard` is added.
- D8 / D9 (phase tables keep the seed's recipe; cost cells rekey): held.
- D3 bound / D4 contract (terminal check unchanged): held.
- Criterion D4 "reason naming both ways forward" against D4's way forward per recipe: **broken** (finding 2).
- Criterion D1/D2 "same edges" against D2/D3 block-exit rewriting: **broken** (finding 3).
- Constraint `PATTERN-refusal-way-forward` against D3's new Expand refusal, D5's two sites and D4's Stuck: held. Each refusal has a criterion with a test that reaches it.

**Findings**

1. MEDIUM, design. Location: `_lym/discussion/decision-record.md:80` (D4's way-forward bullets), with D5 at `:113-115`.
   - Summary: D4 gives Record-Seed's Stuck a single jacquard way forward, the batten sequence `lym batten cancel &lt;slug&gt; --remove` … `lym batten run &lt;slug&gt;`. D5 has already established that a jacquard pair batten did not make needs a different way forward: remove the seed and re-seed with `lym shed seed self --recipe tapestry`.
   - How it is reached: a non-batten jacquard run passes the first-start check and its entry later loses its record. D5 says a re-run over an existing status file "passes over it", and then Record-Seed's Stuck tells it to cancel through batten, which holds no such run.
   - What neither decision states: how the Stuck reason, or the start refusal, tells a pair batten made from any other.
   - Evidence: decision-record.md:78-81 and :113-118. `internal/loomcli/sharedbootstrap.go:105-121` shows the start reads only the worktree's seed.

2. LOW, consistency. Location: `_lym/discussion/decision-record.md:236` (D4 criterion).
   - Summary: the criterion asks for "a reason naming both ways forward". D4 (:78-81) says the reason names one way forward per recipe: the jacquard sequence for a jacquard run, and goto Discussion-Write for a tapestry run.
   - Read literally, a test would assert one message carrying both. The criterion should name each recipe's reason, with its case.

3. LOW, consistency. Location: `_lym/discussion/decision-record.md:232` (D1/D2 criterion).
   - Summary: the criterion says expanded tapestry's rows equal "the Motif block's rows followed by the Jacquard block's rows, in order with the same edges". Expansion rewrites a motif row's empty `on_done` to the block exit Jacquard, and adds the two engine-Block rows. So motif's own rows and tapestry's inlined ones do not have the same edges.
   - The criterion should say it compares the rows as inlined.
   - Evidence: `internal/shedbuild/expand.go:76-79` and `:96-105`; `contracts/recipes/motif-recipe.yaml:7-9`.

**Cross-lens note, not reported as a finding:** D5's criterion "after a done motif run … set to jacquard starts" (:245) depends on the unverified task-branch claim in Notes (:259-261). `ClearMotifRun` discards no branch (`internal/battencli/cleanup.go:137-143`), and `createRefusal` halts on a leftover task branch (`internal/battencli/wire.go:380-394`). That belongs to the handler or bound-breaker.

**Tool calls:** none. My inherited context held everything this lens needed.
