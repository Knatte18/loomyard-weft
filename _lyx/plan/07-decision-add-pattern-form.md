# Card 7 — decision-add-pattern-form

**Edit:**
- `internal/loomcli#decisionInput`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#TestDecisionVerb_AppendsAndCommits`
- `internal/loomcli#TestDecisionVerb_Refusals`

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils/loom/loom-template-arbiter.md`

**Uses:**
- `internal/discussionparser#AddedDecision`
- `internal/discussionparser#PatternChange`

**Intent:**
D5.2: `lyx loom decision add` gains an optional PATTERN-change form, so a PATTERN change decided after the Discussion, by the run's parent, the operator or the run's arbiter, reaches the declaration.

- `decisionCmd` gains three flags, `--pattern-entry <PATTERN-id>`, `--pattern-action adds|changes|removes` and `--pattern-text <new entry text>`, read into new `decisionInput` fields `patternEntry`, `patternAction` and `patternText`; its `Long` help and example name the form.
- `decisionVerb`, after its existing `--by`, empty-flag and caller checks and before the status read, refuses a call that sets `--pattern-text` or `--pattern-action` without `--pattern-entry`, or `--pattern-entry` without `--pattern-action`, with "way forward: pass --pattern-entry and --pattern-action together, and --pattern-text for adds or changes"; the shape of the values is left to the record's own check, which `AppendDecision` re-runs.
  With `--pattern-entry` set it passes `AddedDecision.PatternChange` with the three values, so one call appends the `### Added after Discussion` entry and the bullet in one write, and on a finding (an unknown action, `adds` without text) the record is restored and the refusal names the findings with the existing `decisionWayFindings` way forward.
  The success envelope gains `pattern_change`, the bullet as written, when the form was used.
- `decisionVerb`'s doc comment states the bound: the form appends only a bullet of the record's PATTERN-change shape, through the verb's existing checks and its `--by parent|operator|arbiter` caller guards, so only the run's parent, the operator's own shell or the run's own arbiter declares a change after Discussion, and it removes and rewrites nothing.
- The two stencils of the callers D5.2 admits name the form where they already name the verb: `orch-template-role.md`'s bullet on recording a decision with `lyx loom decision add <slug> --by parent`, and `loom-template-arbiter.md`'s `--by arbiter` line, each adding that a decision that adds, changes or removes a PATTERN entry also passes `--pattern-entry`, `--pattern-action` and, for `adds` or `changes`, `--pattern-text`, as `lyx loom decision add --help` says.

Tests:
`TestDecisionVerb_AppendsAndCommits` gains a row with the three pattern flags whose fake `append` receives `PatternChange` with those values, and whose envelope carries `pattern_change`; its existing rows assert `PatternChange` stays nil.
`TestDecisionVerb_Refusals` gains rows: `--pattern-action` without `--pattern-entry`, `--pattern-text` without `--pattern-entry`, and `--pattern-entry` without `--pattern-action`, each refused before any append with the new way forward; and a fake `append` returning a `pattern-change-malformed` finding, refused naming it with nothing committed.

**ImpactSummary:** one verb gains three optional flags; a call without them behaves exactly as before, and the verb's only engine call stays `AppendDecision`.
