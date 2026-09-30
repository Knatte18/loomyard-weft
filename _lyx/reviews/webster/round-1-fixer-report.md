# Webster round 1 — fixer report

All ten findings are fixed, one commit each, none pushed.
After the fixes, `go build ./...`, `go vet ./...`, `go test ./...` and `go test -count=1 -tags integration ./internal/shell/... ./internal/reedengine/...` all pass.

| Finding | Commit | Change |
|---|---|---|
| F1 | `a0619e016` | panebin.go: file doc, `composePaneLaunchLine`, segment constants, `launchScriptDir`, `stageLaunchScript`, `removeLaunchScripts` and `writeLaunchScript` docs rewritten with semantic line breaks. |
| F2 | `517d5e89a` | spawn.go: `launchStrandLocked`'s pane-binary doc paragraph and the comment above the send rewritten with semantic breaks. The "rather than changing how the pane's shell itself starts" contrast now attaches to the prelude reaching the shell through the sourced script. |
| F3 | `abee19055` | replace.go: the script-cleanup comment now names its real condition (the post-add save failed and the tail failed too), replacing a misleading "Otherwise". |
| F4 | `eb2592d27` | strand.go: `removeStrandLocked`'s new sentence is on its own line. lifecycle.go: `Down`'s launch-dir comment is one sentence per line. |
| F5 | `58c2d8766` | shell.go header, `Source`/`ScriptExt` interface docs, and the posix.go and pwsh.go `Source` docs rewritten with semantic breaks. |
| F6 | `7a31a90db` | doc.go: the changed pane-binary sentences rewritten one per line, with a clause break at `, and`. Untouched legacy lines after them are unchanged. |
| F7 | `f6171dcfa` | Headers and new doc comments in spawn_test.go, panebin_test.go, emptycmd_integration_test.go, launchscript_cleanup_test.go and launchscript_integration_test.go rewritten with semantic breaks. |
| F8 | `890701092` | docs/sandbox-howto.md: the "writing …" participle phrase joined onto its clause's line. The legitimate break before ", so a nested" stays. |
| F9 | `d8a761192` | launchscript_integration_test.go: `readTrimmed` renamed to `readOrEmpty`. |
| F10 | `1b09fdc95` | panebin.go `writeLaunchScript`: the `fail` closure and three inline `os.Remove` calls replaced by one deferred removal that skips once the rename succeeds. |

## Deferred

Nothing deferred.
