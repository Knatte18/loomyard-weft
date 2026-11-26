## Goal

```markdown
Imported from [issue #264](https://github.com/Knatte18/loomyard/issues/264).

## What happened

The first gate run of a batch failed only on a `lym board` process that a `TestLoomPreBootstrap` fixture left running in `internal/loomcli`.
The rerun passed with no change.

## Why it is abnormal

A test leaves a process behind, and the gate fails on it intermittently, unrelated to the change under test.

## What to do

Have the fixture stop, or wait for, every `lym board` process it starts before the test returns.
The relevant fixture is around `internal/loomcli/smoke_prebootstrap_integration_test.go`.

Provenance: task `config-keys-nested`, note `10-loom-key-messages.md`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
