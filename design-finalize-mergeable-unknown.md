# Finalize halts on a pull request whose mergeability GitHub has not computed yet

`finalize-mergeable-unknown` · Task · bug, landing

Imported from [issue #161](https://github.com/Knatte18/loomyard/issues/161).

## What was seen

A task run's Finalize halted with `landing stop: GitHub refused to merge pull request ...: Pull Request is not mergeable` and a way forward naming a second approval or a required check.
Read a minute later, the pull request showed `mergeable: MERGEABLE`, `mergeStateStatus: CLEAN`, no required checks and no reviews; GitHub had not yet computed mergeability (`UNKNOWN`) when Finalize asked.

## Workaround

The parent re-reads the pull request with `gh pr view --json mergeable,mergeStateStatus` and runs `lym loom resume` once it reads `MERGEABLE`.

## Expected

Finalize treats `mergeable: UNKNOWN` as pending: it polls the pull request for a bounded time before merging, and halts only on a definite refusal, naming the actual cause.
