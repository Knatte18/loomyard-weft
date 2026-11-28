# Name config keys as <module>.<path> in Go comments, and guard string literals for bare keys

`config-key-comment-sweep` · Task · enhancement, docs · running · Friction-Reflect

After `config-keys-nested` nests the config keys by area, about 525 Go comment lines in about 199 files, plus single-word keys on top, still name a template key bare.
PATTERN-config-key-names asks that such a comment say `<module>.<path>`.

Sweep every Go comment and package doc that names a config key to its nested `<module>.<path>` form.
A comment-only change, no behaviour or user-visible text.
Deferred from T3 code review round 1, finding F1, to keep that run short.


## From issue #265

Imported from [issue #265](https://github.com/Knatte18/loomyard/issues/265).

## What happened

A code review finding asked for a repository-wide sweep of config key names written without their module, and no tool finds them.
A grep over the underscored template keys was the only practical method.
It missed single-word keys such as `engine` and `scoper`, and any key that ended a string literal, so messages were reported done while several were not.
Self-check forks caught the misses.

An earlier implementer had also found bare key names in messages and help texts outside its card's packages:
`gatecli`, `batcher`, `battencli`, `beamcli`, `webstercli`, `hibernatecli`, `reportcli`, `preflightshed`,
`weaverengine` (`ResolveLadder` and `settingRefusal`), `sheddriver/tackler.go`, `shedarbiter`, `burlerengine/fix.go` and `loomshed/plancontext.go`.
Some of these may have been fixed since.

## Why it is abnormal

A message or help text naming a key without its module can regress at any time, and nothing detects it mechanically.

## What to do

Add a guard test that scans Go string literals for a config template's key names written without their module.

Provenance: task `config-keys-nested`, notes `burler-code-r1.md`, `11-module-key-messages.md`.
