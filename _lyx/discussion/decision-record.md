# Decision record — discussion check refuses an over-cap PATTERN text

## Goal

A decision record whose `## PATTERN changes` bullet declares an `adds` or `changes` text too long for a PATTERN.md entry line fails the discussion check, so the overrun is caught at Discussion or at `lyx loom decision add` instead of first at Plan-Write.

## Scope

- `internal/discussionparser`: `Validate` gains one check over the parsed `## PATTERN changes` bullets.
  It measures the overview line each `adds` or `changes` bullet prescribes, `` - `PATTERN-<id>` — <text> ``, in runes against the PATTERN entry line cap, and reports one finding per bullet over it.
- `lyx loom decision add --pattern-text` refuses an over-cap text through the same check, since `AppendDecision` runs `Validate` after the write and restores the record on a finding.
- The discussion format spec, `contracts/specs/loom-discussion-spec.md`, states the cap in its PATTERN changes section, and the package doc names the check.
- Out of scope: the plan gate's `pattern-entry-line-cap`, the PATTERN format checker, and the cap's value.

## Decisions

### The check lives in `Validate`, apart from the bullet shape check

Decision: a new finding, `pattern-change-over-cap`, comes from `Validate` after `ParsePatternChanges`, and `ParsePatternChanges` stays a shape check.

Rationale: `Validate` is the one function the loom discussion gates, the darn gate, `lyx loom validate-discussion` and `AppendDecision` all run, so one site covers Discussion-Write and `lyx loom decision add`.
Keeping the length out of `ParsePatternChanges` leaves `DeclaredPatternChanges`, the plan gate's mismatch check and the dropped-bullet check reading an over-cap bullet as a well-formed one rather than refusing a record the discussion gate already judged.

### The measured line is the overview line the bullet prescribes

Decision: the check measures `` - `<ID>` — <text> ``, the line PATTERN.md carries for the entry, in runes, with the bullet's text trimmed and without any appended-after-Discussion suffix.

Rationale: Plan-Write must fence exactly that line, and `pattern-entry-line-cap` measures it in runes against `pattern.MaxEntryLineChars`, so the two checks agree on every bullet.

### The cap is a package constant pinned to the PATTERN cap by a test

Decision: `internal/discussionparser` declares its own unexported cap constant, and an in-package test asserts it equals `pattern.MaxEntryLineChars`.

Rationale: the package's leaf test admits no non-stdlib import in production code, and `internal/pattern` pulls the stencil packages in; the test catches a drift of either value.

### The finding applies to appended bullets too

Decision: a bullet carrying the appended-after-Discussion suffix is measured like any other, and the finding's way forward names `lyx loom decision add` re-recording the entry with a shorter text for such a bullet.

Rationale: `AppendDecision` must refuse an over-cap `--pattern-text`, which it does only when `Validate` measures the appended bullet; a re-record replaces the bullet in place, so a parent can always clear the finding.

## Constraints

- `PATTERN-sole-parsers`: the record's format stays read only by `internal/discussionparser`.
- `internal/discussionparser` production code imports only the standard library, per its leaf test.
- `PATTERN-refusal-way-forward`: the finding's detail names its way forward, and a test reaches it.
- `PATTERN-gate-self-check-parity`: the gate and `lyx loom validate-discussion` keep calling the one `Validate`.

## Auto-mode assumptions

- The cap counts runes, as `pattern.Check` and `pattern-entry-line-cap` do.
- A `removes` bullet prescribes no line and is never measured.

## Open risks

- A record already holding an over-cap appended bullet fails the discussion check until its parent re-records the entry; no live run is known to hold one.

## Acceptance criteria

- `Validate` reports one `pattern-change-over-cap` finding at the record's path for an `adds` and for a `changes` bullet whose overview line is one rune over the cap, naming the entry, the length and the cap, with a way forward.
- A bullet whose overview line is exactly at the cap passes, and a line of multi-byte runes is measured in runes.
- A `removes` bullet is never measured.
- `AppendDecision` with an over-cap `PatternChange` returns the finding and leaves the record's bytes unchanged.
- The cap constant equals `pattern.MaxEntryLineChars`.
