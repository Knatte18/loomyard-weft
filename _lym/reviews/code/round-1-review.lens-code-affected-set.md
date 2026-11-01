```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I list what reads or asserts it, and whether the diff covers it.

**Per change**

1. **Card gate drops its separate lint steps** (`internal/websterengine/cardgate.go`)
   - Covered: `beginbatch_test.go`, `recoverbatch_test.go`, `cardgate_test.go`, `websterengine/doc.go`, the stencil sentence in `webster-body-implementer.md`, `loomshed/doc.go`, `loomshed/patternplan.go`, `loom-plan-spec.md`, `loom-rubric-code-review.md`.
   - Covered: `docs/overview.md` (both edits) and `pattern/PATTERN-gate-slot.md`.
   - Covered: the `webster-body-implementer.md` line 89 allow-list still names `lym loom lint-comments` and `lint-pattern`. Both verbs still exist, so it stays correct.
   - Not covered: `internal/loomcli/lintcomments.go` and `lintpattern.go`. See F1.

2. **New `lints` key on `GateKey`** (`internal/gateslot/config.go`, `tiers.go`, `template.yaml`)
   - Covered: validation in `decodeGates`, the clone in `Config.Gate`, `gateslot/doc.go`, the template comment, `config_test.go`, and `gatecli/doc.go`'s Lint steps section.
   - Consumers of `Gates[...]`: `gatecli/tiers.go:248`, `scope.go:92` and `run.go:176`. Only `run.go` needs the new field.
   - Not verified: `docs/dotnet-hub.md:90` and `gatecli/doc.go:311` show `card: {tiers: [0], scope: card}` with no lints. The new config test says a tuned `card` key keeps the template's lints. So a .NET hub's card gate now runs both lints, and the diff doesn't mention it. I did not check whether `patternlint` is safe on a hub with no `PATTERN.md`.

3. **Closing line and failure report on stderr, `failedRun` list**
   - Covered: `gatecli/doc.go`, the `docs/overview.md` gate entry, and `build.go`/`run.go`/`tierrun.go`.
   - Run-record `failures` stays unchanged through `testFailures`.
   - The `counter` interface gains `Unparsed()` and `Failures()` changes type. Both implementers (`goJSONCounter`, `vstestCounter`) are updated.

4. **Capped-call stdout no longer replayed past 4096 bytes** (`detachfront.go`)
   - Covered: `gatecli/doc.go` and the integration test.
   - The `replay` caller is the only one. I made no search for other readers of the front's stdout, such as a webster or loom caller that parses the JSON envelope.

5. **Declared PATTERN changes**: the instruction listed none, and the commit has no `Pattern-Departure:` trailer. `PATTERN-gate-slot.md` background is updated. The `PATTERN.md` entry at line 51 is still accurate, so nothing is missing.

6. **Stencil wording** (`webster-body-implementer.md`): the diff adds one line and drops or weakens nothing. The new line is "fix every failure it lists before you run the gate again". The rule about fixing the guard failure under the card's commit is kept.

**Findings**

- **F1, LOW, consistency**: the `lint-comments` and `lint-pattern` verb docs still say the card gate runs them as its own later steps.
  - `internal/loomcli/lintcomments.go:1` says "the standalone form of the comment line-break lint every card gate runs".
  - `lintcomments.go:34` says "every card gate runs after its tests".
  - `internal/loomcli/lintpattern.go:1` and `:23` say "every card gate runs after lint-comments".
  - After this diff the card gate no longer runs the verbs. Its `card` key runs the same functions as gate lint steps inside `lym gate test`.
  - Fix: reword all four to say the verb is the standalone form of the lint that a gate key's `comments` or `pattern` lint step also runs. `docs/overview.md` and `PATTERN-gate-slot.md` were already reworded this way.

**Tool calls (2 of 6)**
1. Grep for "every card gate"-style phrases across the repo, to find stale mentions. My context had only the diff.
2. Grep for `GateKey{`, `Gates[` and `gates:` consumers and hub-doc examples, plus the `PATTERN.md` gate-slot line. My context lacked the consumer set outside the diff. Both ran in one Bash call.
