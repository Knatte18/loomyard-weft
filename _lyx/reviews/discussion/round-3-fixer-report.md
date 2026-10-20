# Fixer report — discussion round 3

## Changes

- F1, fixed as proposed: D1's no-tree refusal and the editor sentences say non-empty `tree_scope_verify`.
  D1 now accepts both keys empty, as the template ships them.
  A new criterion pins that both keys empty load with scope off, and that `--set tree_scope_verify=` is accepted with no tree.
  The no-tree criterion and the editor-save criterion say non-empty.
- F2, fixed differently: D1 now says the editor load refusal gets its own sentinel, checked the way `errHubFileChanged` is, so the staging copy is kept.
  The editor-save criterion pins the kept copy.
  The proposed way forward, re-apply from the staging copy and never re-open, was not made.
  Only the editor can remove a key, and every open of the editor overwrites the staging copy (`internal/configcli/hubwide.go` `editHubWide`, the `os.WriteFile(staging, base, …)` before `EditPath`).
  So the message says to copy the staging copy out of its path, re-open the editor and re-apply the edit from that copy.
  The last-tree clause naming `--set tree_scope_verify=` first is kept.
- F3, fixed differently: the malformed-key refusal names the key as written, and the duplicate refusal names both keys.
  Those refusals carry no way forward in `LoadConfig`'s text, because `newLoadFailure` (`internal/configcli/configcli.go:265`) wraps the loader text and appends its own `--set` way forward.
  Each caller adds its own way forward: the editor at run start, the editor refusal's way forward at an editor save, and `newLoadFailure`'s `--set` one for a `--set` write, which for a duplicate means the spelling already in the file.
  The rollback sentence is bounded to a write after which `LoadConfig` fails with a new error, matching `newLoadFailure`'s before-and-after comparison.
  The duplicate-key criterion now pins that both keys are named.
- F4, fixed as proposed: D2 classifies through a new `fabricengine` read beside `ChangedPaths`, run with `-z` and `--no-renames`.
  `ChangedPaths` and its other readers keep their output.
- F5, fixed differently: D5 says the marker kinds name the line, and the parse kind names it only when the parser reports one.
  An unreadable test file arrives as the parse kind with the OS error.
  `GuardScanError` carries no kind and formats the cause with `%v` (`internal/impactset/guards.go`), so the verb cannot tell an unreadable file from a parse failure.
  The parse kind's single way forward is therefore "make the file readable or fix it until it parses", and the proposed separate "make the file readable" way forward was not made.
  The guard-scan criterion names the file for an unreadable test file.
- Support log: added a Round 3 entry under `## Review rounds`.

## Self-check

- fix-claims, LOW: the `--set` rollback sentence was unbounded when the file already fails with the same error. Fixed as proposed (bounded to a new load error).
- fix-consistency, LOW: the support log had no Round 3 entry. Fixed as proposed.
- fix-consistency, LOW: the duplicate-key criterion did not pin naming both keys. Fixed as proposed.
- fix-consistency, LOW: the editor save got two ways forward, and "the file already holds the key" was false at an editor save. Fixed as proposed (run start names the editor, and an editor save defers to the editor refusal).
- fix-proposal and fix-way-forward, MEDIUM (merged): "re-apply from the staging copy, never re-open" cannot be done for a key removal. Fixed with the copy-out-then-re-open way forward (see F2).
- fix-proposal and fix-way-forward, MEDIUM and LOW (merged): a `--set` key refusal would carry both the loader's editor way forward and `newLoadFailure`'s `--set` one. Fixed as proposed (`LoadConfig`'s key refusals carry none, and each caller adds its own; a duplicate's `--set` way forward names the existing spelling).
- fix-proposal and fix-way-forward, MEDIUM (merged): the verb cannot tell an unreadable test file from a parse failure. Fixed with the combined parse-kind way forward (see F5).

## Disputed

None.

## Deferred

None.
