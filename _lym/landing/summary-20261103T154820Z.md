# Burler review runs an assistant reviewer seat beside the chair

A Burler review round can now run an extra reviewer, the assistant, on any engine its model-spec names, Codex first.
The chair waits for the assistant's report, consolidates it as one more report and keeps the verdict, so the assistant never gates a round.
Every assistant key defaults to empty, so no round runs one until the operator sets it.
This is part 4 of the codex-engine project: decisions D1, D2 and D3 of its record, on top of the codexengine, skills, shell guards, engine blocks and heddle fallback the earlier parts landed.

## The assistant seat kind (`internal/seatengine`)

A table now holds advisors only, writers only or assistants only after the lead.
An assistant writes its outputs and nothing else.
It gets no up line and no mail, sends none, and its held turn end stays quiet.
`seat-directive-assistant` tells it to ignore `[creel]` text in its pane.
The lead of an assistant table includes `seat-directive-assistant-chair`, and `Table.Validate` refuses a table that misses either directive, mixes kinds, or gives an assistant no end marker or a shared one.

Each assistant carries an `EndMarker`, which the engine writes once the run ends, whatever the ending, or when it does not start.
The marker text names the ending:

- `done`, written only after the seat gate passed;
- `gate failed`;
- `died`, `timeout` or `stopped`;
- the wait error;
- `not started: <reason>`;
- `not found`.

A fresh run removes stale markers and outputs before the chair starts.
An assistant does not start, and the step goes on without it, in three cases:

- its `SkipReason` is set;
- its start fails;
- its engine lacks an engine block its stencil includes.

`Validate` and `CheckMarkers` pass that missing block over for an assistant alone.
The chair's hold deferral counts an unended assistant that reads mid-turn.
Every assistant is stopped when the chair ends.
On resume, an assistant the probe did not find keeps the marker its run wrote, or gets `not found`.
The chair sees each assistant's report and marker through the new reserved marker `assistants`.
The assistant's parent directive is the no-parent variant.
Its edit directive is its engine's own copy, read by the new `editdirective.EngineDirective` from `engine/<engine>/edit-directive.md`.

## The review half (`internal/burlerengine`)

`RunOpts.Assist`, `AssistFan`, `AssistKey`, `AssistTimeout` and `AssistSkip` name the round's assistant.
When `Assist` names a model, the review table gains a seat named `assist`, so its role is the review role followed by `-assist`.

The chair runs `burler-template-reviewer-assisted`.
Its review step renders `burler-assist-consolidation` through the new optional `assist_rules` marker, which tells the chair to:

- wait on the marker;
- judge a done assistant's findings as a fork's;
- tag kept ones `origin: assist:assist`;
- reject false ones in `## Rejected`.

A round with no assistant renders the same stencil and the same review-step bytes as before.
The assistant runs `burler-template-assist-reviewer` with its engine's `assist-reading` block, the benches' block v2 with the yardstick bullet.
It reads its own explore and review steps, composed for its own engine and report path, with the chair's fasit, rubric, PATTERN directive, focus and prior rounds.
A fan runs through `burler-assist-cluster-fan` and the engine's `assist-fan` block.
Its gate is the review-parse entry on its report.

The report and marker sit under the anchor's `.lym` at the review path's subpath.
Their names carry the review stem, the attempt token and the seat, so a retry never shares the first attempt's files.
Once the chair ends, fresh or resumed, a report whose marker reads `done` is copied beside the review as `<review stem>.assist-assist.md`.
`ReviewResult.Assists` reports the outcome; no verdict or finding reads it.

## The review row (`internal/shedadapters`)

`BurlerHalfDeps.Assist` carries the segment's per-round assistant models and fans, their keys, and the capped timeout and skip reason.
Before each attempt spawns, the row writes three things into the round's usage record:

- the attempt token, `<round>`, or `<round>b` before the retry;
- the assistant's model choice;
- its fan.

`probeLiveReview` builds the whole table, chair included, from that record, so a probe after lym dies mid-retry finds the retry's chair and assistant.
A record without the token reads as the bare round with no assistant, whatever the config says now.
The copy beside the review joins the files archived before a retry or a new round.

## Config (`loom.yaml`, `tapestry.yaml`)

New keys:

- `discussion_assist_reviewer_model`, `code_assist_reviewer_model` and `plan_assist_reviewer_model`: a model-spec or a per-round list, where an empty entry runs no assistant that round.
  A non-empty entry that is no model-spec is refused, naming the module, the key and the entry.
- `<segment>_assist_reviewer_fan`: the assistant's fan in the fan-list grammar.
- `assist_reviewer_timeout_min`, 60 by default.

`loomengine.CapAssistTimeout` caps the timeout so three things end before the chair's timeout: the assistant's start with its start retries (the new `shuttleengine.Config.StartAllowance`), its run, and one await cap.
The chair's timeout is `review_timeout_min`, or `run_timeout_min` when that key is 0.
A cap logs a Warn naming both keys.
A cap that leaves no time gives a skip reason, which the marker carries.
A `review_timeout_min` of 0 still loads.

## Stencils

The new stencils are:

- `seat-directive-assistant` and `seat-directive-assistant-chair`;
- `burler-template-reviewer-assisted` and `burler-template-assist-reviewer`;
- `burler-assist-consolidation` and `burler-assist-cluster-fan`;
- the engine blocks `engine/{claude,codex}/{assist-fan,assist-reading,edit-directive}`.

`burler-step-review` gains the optional `assist_rules` marker.
`TestStencils_NoEditRuleRestatement` exempts the engine copies of the edit directive.

## Design decisions

- **No fork authorization for the assistant.**
  The assistant seat does not set `ForkSubagents`, because codexengine does not meet `RequirementForks` and lym's fork machinery for Codex is deferred.
  A fanned assistant spawns through its engine's own subagents, as its `assist-fan` block says: `spawn_agent` with `fork_turns: "all"` on Codex, general-purpose Agent subagents on Claude.
- **Assistant edit directive from an engine block.**
  The assistant's edit directive is a whole engine block (`engine/<engine>/edit-directive`), not the shared `edit-directive` stencil.
  A Codex session has `apply_patch` and shell reads, not Read, Edit and Write.
- **Copy at conclusion.**
  The copy beside the review happens in `burlerengine`'s `concludeReview`, which both the fresh and the resumed path reach.
- **Attempt recorded every round.**
  The row records the attempt token for every round, assisted or not, so the probe also finds a plain retry's chair.
- **One timeout cap per run.**
  The cap is computed once per run in `internal/loomcli`'s wiring, where shuttle's config is loaded.
  Its await cap is the keep-warm cap of role `burler-review`, whatever role prefix a segment's row sets.

## Open items

- **Weft write unproven.**
  The record (D6, Open risks) wants the build to prove that the Codex sandbox lets the assistant write through the `.lym` link into the weft writable root before building on D2.
  This change was built at a desk with no Codex run, since Codex runs are the operator's and capped, so that write is still unproven.
  If the sandbox blocks it, the assistant ends `gate failed` or at its timeout, and the chair goes on without it.
- **Smoke test not run.**
  The llm-tier smoke test of a Codex assistant on `luna[low]` is the operator's to run, capped at one run.
