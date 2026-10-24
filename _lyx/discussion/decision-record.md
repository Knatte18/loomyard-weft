# Decision record — restore to PATTERN the moved rules that bind callers outside their package

## Goal

Put back in front of every spawned agent the moved PATTERN rules that code outside the owning package can break with no test to catch it.

## Scope

- Restore four rules to `PATTERN.md` as one-line entries: `sparse-hub-base`, `parent-lock`, `agent-name` and `wait-mark-display-only`.
- Restore the two leaning-back rules too: `transient-stop` and `friction-capture`.
- Each rule's detail stays in its owning package's doc, which cites the restored ID.
- `friction-capture`'s clause that `lyx loom start` vouches for the resume it spawns goes back into `internal/friction/doc.go`.
- Out of scope: tripwire tests for the restored rules, background files under `pattern/`, and the three rules the audit judged fine (`hub-wide-config`, `recipe-owns-failures`, `batcher-registry`).

## Decisions

### Restore all six

Decision: restore the four the audit names and the two leaning back.
Rationale: `transient-stop` has no scan of mark sites outside the package, and `friction-capture` binds every new halt or refusal path outside `internal/friction`, so both bind outside code with review as the only guard, the same test the four meet.

### Overview lines only, no background files

Decision: each restored entry is an overview line with no `[background]` link.
Rationale: the owning package doc already holds the rule's bounds and enforcement, and the board entry asks to keep the detail there; a background file would hold a second copy.

### Each line names the owning package

Decision: each entry line names the package whose doc holds the detail, and that doc's section cites the entry's `PATTERN-` ID.
Rationale: the format checker admits only a `pattern/` link, so the package name is the overview's pointer, and the ID in the doc ties the two together for a grep.

### Each entry goes under the topic heading its subject fits

Decision: `sparse-hub-base` under CLI and config, `parent-lock` under Fabric and git, `transient-stop` and `friction-capture` under Shed and loom, `agent-name` and `wait-mark-display-only` under Agents and prompts.
Rationale: the headings group by subject, and each rule sits beside the entries an agent touching that subject reads.

## Constraints

- `PATTERN.md` stays under the `internal/pattern` caps: each entry line at most `MaxEntryLineChars` characters and the overview under the headroom line.
- An entry marks `(test)` only where a test enforces the rule, and each restored entry keeps the marking its removed line carried.
- Semantic line breaks in every edited comment and markdown file.

## Auto-mode assumptions

- A darn run has no operator, so the two leaning-back rules are decided by this record rather than asked.
- The removed lines at `04aafa88c^` are the starting text, tightened where the package doc now states the rule more precisely.

## Open risks

- The overview grows by about 1 KB, paid in every spawned agent's prompt; it stays well under the headroom line.

## Acceptance criteria

- `PATTERN.md` holds the six entries, and `internal/pattern`'s real-file check passes.
- Each owning package's doc section cites its restored ID.
- `internal/friction/doc.go` states that `lyx loom start` vouches for the resume it spawns.

## PATTERN changes

- `PATTERN-sparse-hub-base` — adds: Writing a hub base file, from any package: only the units the hub tuned, never a template fill; only `internal/hubreconcile`'s one-time strip drops a template-equal unit.
- `PATTERN-parent-lock` — adds: Moving, merging into or landing on a local parent branch: only in a run's Finalize, under `internal/parentlock`'s lock for that parent.
- `PATTERN-transient-stop` — adds: Marking a failure transient: only `internal/shedengine`'s mark, translated by `internal/shedtransient` at a producer or step-bootstrap boundary; never on a gate verdict.
- `PATTERN-friction-capture` — adds: Halting or crash-resuming a loom run, or refusing in webster: Go writes a note via `internal/friction`; none is archived before a reflection covers it. (test)
- `PATTERN-agent-name` — adds: Forming, parsing or matching an agent name: only `internal/agentname`, a strand's full name formed once by reed; a spawned role is its module's constant.
- `PATTERN-wait-mark-display-only` — adds: Marking a pane waiting: only reed's `SetWaitMark`; no Go decision reads `@lyx_wait` or `@lyx_wait_start`.
