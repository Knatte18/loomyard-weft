# Name config keys in full in Go comments; guard literals for bare keys

`PATTERN-config-key-names` asks that every message, comment and doc name a config key in full, `<module>.<dotted path>`.
Messages and help text were brought in line when the loom and tapestry keys were nested,
but hundreds of Go comments and package docs still named keys bare, such as `beam.yaml's pull_request`,
`shed.yaml's driver_kind`, `review.max_bounces` or `tier_parallel`,
and nothing caught a message that regressed.

## What changes

- A new tier0 guard, `TestKeyNames_StringLiteralsNameEveryConfigKeyWithItsModule` in
  `internal/configreg/keynames_enforcement_test.go`, fails on a production string literal under `cmd/` or `internal/`
  that names a template key without its module.
  It reads every registered module's template and indexes each key path and each trailing part of it,
  so both `max_bounces` and `code.max_bounces` count as bare forms of `loom.code.max_bounces`.
  A failure names the file, the line, the bare name and every full name it can stand for.
- Production Go comments and package docs across the module now name config keys in full,
  for example `beam.pull_request`, `shed.driver_kind`, `loom.review.max_bounces`, `gate.tiers.<tier>.timeout_min`,
  `drift.capture.timeout_min` and `claude.accounts`.
  This includes one-word keys a name match cannot find, such as `gate.scoper`, `shuttle.engine`, `weaver.ladder`
  and `reed.watchdog`.
- The `lym config` help names `shuttle.deny_python` in full, rendered through `configengine.KeyName`.
  This is the only literal the guard found naming a key bare.
- `PATTERN.md` marks `PATTERN-config-key-names` as test-enforced.

## Design decisions

- The guard checks string literals only, not comments.
  A comment scan with the same rules finds several dozen words that only share a key's name:
  a recipe row's `max_bounces`, `driver_kind` or `verify_source: tree_scope`, a board entry's retired `pull_request` field,
  a drift step's `timeout_min` field, a prompt marker named `rename_tools`, and GitHub's `pull_request` rule type.
  Keeping those out of a comment guard would take an allowlist about as long as the findings.
- What the guard skips, so that data and qualified names pass:
  - struct tags and import paths;
  - a literal that is one key name whole, such as a map key or a logger attribute;
  - the arguments of a call to a function whose name ends in `KeyName` or `keyName`;
  - a name right after `.` or `%`, which is qualified when the text is rendered;
  - a name after `--set` or `--unset`, whose argument is a path inside the module the command names;
  - a name followed by a colon, a field of a document example.
- A one-word key counts only in the form "the `<name>` key".
  Words such as `engine`, `model` or `beam` are ordinary prose in many messages, and a wider match reported tool-call parameters and recipe names.
- Seed-only modules (`models`, `profiles`) are not indexed, since their keys are names the operator chooses.
  An open or per-name map is indexed as a key, but its entries are not.
- An allowlist in the test holds nine literals that write a key's name for something else.
  These are recipe fields, the run-log line format that `gateslot` writes and `verifytree` parses back,
  a path that the callee qualifies through its own key-name helper, and the Codex CLI's own `sandbox_mode` option.
  Each entry states its reason, and an entry that matches nothing fails the test.
- Comments that name a group of keys rather than one stay as they are, such as "the driver, tackler and arbiter keys".
  So do comments where no single module fits: a review segment's `reviewer_model` lives in `loom` for most segments
  and in `tapestry` for the plan segment.
- Where a full name pushed a comment line past the length limit, the line was split into two sentences or broken at a clause boundary.
  A few comments gained or lost a word in the split; none changed meaning.

## To check by hand

- Comments that use a key's name for a recipe field, a board entry field or a drift step field were left bare on purpose.
  A reviewer may still find a comment where the sweep picked the wrong module or missed a one-word key.

## Merge danger

**Door:** two-way
**Blast radius:** comments
