# Claude settings golden has no update flag

`settings-golden-update-flag` · Note · enhancement, claude, gate

Imported from [issue #257](https://github.com/Knatte18/loomyard/issues/257).

## What happened

The Claude settings golden, `internal/shuttleengine/claudeengine/testdata/settings-golden.json`, has no update flag.
`TestBuildSettings_MatchesGolden` only compares against it.
Each change to a shellguard steer or pattern needed a temporary test file to render the new golden.

## Why it is abnormal

Every guard-row change rewrites this golden, and each agent making one has to write and then remove its own renderer.
That costs turns, and a leftover or wrong renderer is a risk to the commit.

## What to do

Add an update mode to the golden test that rewrites the file from the rendered settings and deny notice, reachable through `lym gate test`.

Provenance: task `reed-client-and-bus-guard`, note `burler-code-r1.md`.
