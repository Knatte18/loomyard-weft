# Webster fork and pattern-directive fixes from the stencil audit

Fixes the five Webster fork and pattern-directive findings of the stencil intent audit.

## What changes

- **Rename uses the hub's tool.**
  `language.yaml` gains an ordered `rename_tools` key, validated like `verify_tiers`.
  The Webster implementer body renders it, and `edit-directive.md` states the one exception it licenses: a Webster implementer on a `Rename` card's symbol pair runs the listed tool.
  The plan spec's `Rename` row and `## Rename mechanic` now separate file pairs (`git mv`, then surgical edits) from symbol pairs (the hub's tool, then the grep pass).
  A new plan-gate check, `rename-tool-missing`, flags a symbol `Rename` in a hub with an empty list.
  `begin-batch` and `recover-batch` refuse the same case with `{"rename_tool_missing": true}`, and the master stencil gains a rung for it.
- **A fork's PATTERN departure travels in its commit.**
  A fork, recovery strand or verify-fix fork records a departure as a `Pattern-Departure: <entry-id> — <what differs and why>` trailer.
  `record-batch` and `recover-batch` read the trailers of the batch's own first-parent non-merge commits into a new digest field, `departures`.
  The next fork's prior-batch line and the Master's progress trail carry them.
  The verify gate records each fixer commit and its trailers, and Go appends them to `summary.md` as `## Verify gate fix commits`.
  The Webster-Review rubric reads commit messages and judges a departure on its cited entry.
- **Spawn and completion wording.**
  The fork prefix, Master template and verify-fix body now match how Webster runs: the fork's turn ends on its report, and the Master keys self-identification on "You are an implementer fork".
  A batch-gate failure the batch's cards did not cause reports `OK` when the plan declares a `## verify:` section (new `plan_verify` marker), and stays `FAILED` when it declares none.
- **Inline kick-start pack layout removed.**
  `kickstart_pack_file` is retired through `RetiredKeys`, so a hub file holding it still loads and reconcile prunes it.
  `kickstart_pack_per_card: true` still selects the per-card layout; every other pack uses the pack-file layout.
- **Declared PATTERN changes.**
  The decision record and the plan overview gain an optional `## PATTERN changes` section (`adds`, `changes` or `removes` per entry), checked by `discussionparser` and by the new plan-gate check `pattern-changes-mismatch`.
  `lyx loom decision add` can append a bullet after Discussion.
  Every pattern directive renders the declared changes through a `pattern_changes` marker and judges a listed entry by its declared text.
  The Webster-Review rubric flags a declared entry the `PATTERN.md`/`pattern/` diff does not land as BLOCKING.

Docs follow in the same change: the `doc.go` files of `languageconfig`, `websterengine` and `pattern`, the discussion, plan and refusal specs, and the config reference docs.

## Design decisions

- `internal/pattern` stays a leaf: each `pattern.Directive` caller reads the record through `discussionparser` at a path its geometry is told, and passes the entries in.
  A record that fails to parse refuses the spawn, with the operator's fix and `lyx loom validate-discussion` as the way forward.
- The rename exception is bounded in prose only; no audit checks which command a fork ran, and the shell guard still applies to the tool.
- `departures` is informational and never changes a batch's status, like `deviations`; malformed trailers are carried verbatim.
- `CLAUDE.md` gets no declaration, and this change edits no PATTERN entry.

## To check by hand

- This hub's `language.yaml` has no `rename_tools` yet.
  Until the operator sets it (`lyx config language` from the prime), a plan here with a symbol `Rename` fails `rename-tool-missing`.
  A Python-based tool also needs shuttle's `deny_python` off.
- Review left open: the `## Verify gate fix commits` heading is written by Go, while `contracts/specs/webster-spec.md` and `contracts/specs/loom-final-summary-spec.md` still describe the old Master-written `## Verify gate fixes` section.
  Check the specs and the master stencil agree on one writer and one heading.
- Review left open: the Bouncer's refusal row in `refusal-spec.md` may misstate the outcome of a refused spawn.
  Check it against `internal/shedadapters/bouncer.go`.
- Review left open: the `pattern_changes` bound (a declaration covers only listed entries, only through bullets of the specified shape, appended only through `decision add`'s caller guards) may be stated by no stencil or spec.
  Check it appears where the directive renders.
- Review left open: test helpers in the `internal/loomcli`, `internal/loomshed` and `internal/shedrecipe` tests may be re-signed inconsistently across call sites.
  Check the callers of the changed helpers.
- The verify gate's first attempt failed on help-text length (`lyx loom decision add`, `lyx loom validate-plan`) and an empty `PatternRecordPath` in `envkit`; the fix commit `e60af7b0b` cleared both.
  The fixer ran the touched packages' unit and integration tests and `lyx loom lint-comments`, not the full `lyx gate test ./...` and `lyx gate test --tags integration ./...`; run both before pushing.
