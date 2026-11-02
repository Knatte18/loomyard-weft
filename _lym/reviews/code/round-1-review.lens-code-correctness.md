```yaml
lens: code-correctness
outcome: reported
```

**Task: code-correctness lens. I ran each function or branch the diff changes with one concrete input and report whether it was correct.**

## Per-function list

| Function / branch | Case run | Result |
|---|---|---|
| `Summary`, envelope detection | `{"ok":true}`, `{"slug":"x"}`, `[1]`, `{"ok":true} {"ok":true}`, `{"ok":true}x` | Correct. The first gives `ok\n`. The rest are rejected by the `isBool` check or `decoder.More()`. |
| `Summary`, failed with empty error | `{"ok":false}` | Prints `failed: ` with a trailing space. Cosmetic only (NIT). |
| `Summary`, `ok:true` with an `error` string | `{"ok":true,"error":"x"}` | The top-level loop skips the `error` key (`summary.go` field loop), so the text is dropped silently. Unlikely in practice. Reported as F3. |
| `Summary`, numbers | `{"count":0}` | `json.Number` is not matched by `isEmpty`, so `count: 0` prints. The summary doc says empty values are left out, but a zero number is not empty. Acceptable. |
| `Summary`, plain list of 6 | `"tips":["a".."f"]` | `tips: 6 items`. Correct and matches the test. |
| `Summary`, plain list holding `null` or a newline | `[null]`, `["a\nb"]` | `null` prints as `<nil>` through `fmt.Sprint`. A newline in an element breaks the one-line layout. Cosmetic (NIT). |
| `summaryRenderer.list`, note key with a long plain list | `{"ok":false,"warnings":["1","2","3","4","5","6"]}` | **Wrong.** `list` collapses it to `warnings: 6 items` and never calls `notes`, so no warning is printed. Reported as F1. |
| `summaryRenderer.list`, `way_forward` or `*_error` as a list of 6 | same path | **Wrong**, same defect as F1. |
| `isNoteKey`, plural `errors` | `{"errors":[...6]}` | Not a note key (only `error`, `*_error`, `*warning(s)` match), so it collapses to a count with its text lost. Same family as F1. |
| `field`, depth boundary | map at depth 2 | Collapses to `N fields` and `notes(depth+1,"",…)` still prints the notes. Correct and matches the `DeepObject` test. |
| `notes`, path for notes found inside a collapsed object | `tips.t.error` | Correct. The path starts below the collapsed object, so `warnings: diverged` has no parent label. Readable enough. |
| `noteLines`, note whose value is an object | `"way_forward":{"a":1}` | Prints `way_forward: 1 fields`, so the content is lost. Edge case, no known producer (LOW). |
| `item` | object with label and line, object with neither, non-object element | Correct. The `hid` flag is set only for non-shown, non-empty, non-note keys. |
| `SummaryWriter.Write`, envelope split across writes | `"{\"ok\":"`, then `"true}\n"` | Correct. It holds, then summarises on the newline. |
| `SummaryWriter.Write`, prompt with no newline, then `{` | `"prompt: "`, then `{...}\n` | Correct. `atLineStart` is false after the prompt, so the `{` text passes through unchanged. |
| `SummaryWriter.Write`, several lines in one write | `"{..}\nx\n{..}\n"` | Correct. The loop advances `p` and `holding` resets in `writeHeld`. |
| `SummaryWriter.Write`, multi-line indented JSON | `"{\n"`, then body lines | `{` fails `Summary` and is written as is. The body lines pass through unchanged. No bytes lost, duplicated or reordered. |
| `SummaryWriter.Write`, underlying writer fails on a held line | error in `writeHeld` | `held` and `holding` are reset first, so no stale state. The count `total-len(p)` includes the held line, so it counts bytes as consumed that were not delivered. Acceptable. |
| `SummaryWriter.Write`, underlying writer fails on a pass-through chunk | partial write | Returns bytes before the chunk, so a partly written chunk is under-reported. Harmless. |
| `SummaryWriter.Flush` | nothing held; an unterminated envelope held | Correct in both. The held line is summarised, or written as is when it is not an envelope. |
| `SummaryWriter`, a line that never ends | `{` followed by unbounded data | Held in memory with no cap until `Flush`. Only matters for a very large `{` line. LOW at most, not reported. |
| `UseReadableOutput`, flag order | `--envelope` set; role audience; verb's own `--json`; non-terminal | All return before `SetOut` and match the test rows. The persistent flag is merged into `cmd.Flags()` after parsing, so the `Lookup` finds it. |
| `UseReadableOutput`, `onFinish(cmd.Context(), …)` with no exit state | cobra executed outside `RunRootCtx` | `onFinish` is a no-op, but `SetOut(summary)` has already run. An unterminated held line is then never flushed and is lost. Every production path goes through `RunRootCtx`. Reported as F2 (LOW). |
| `RunRootCtx`, order | `err != nil` with `es.finish` steps | The finish steps run before the cobra error envelope, which goes to the raw `out` and is never summarised. Order is correct. A cobra-level error therefore prints the raw one-line JSON error envelope on a terminal. That is consistent with the summary rules (no summariser is installed on that path) and not a defect. |
| `RunRootCtx`, nil exit state | `NewExitContext` always returns a non-nil `es` | Cannot happen. Correct. |

## Findings

**F1: MEDIUM, design.** `internal/output/summary.go`, `summaryRenderer.list` (the plain-list branch with `len(items) > summaryListLimit`), reached from `field`.
- A long list under a note key (`warnings`, `*_warnings`, `way_forward`, `*_error`) collapses to `key: N items`, and `notes` is never called on it.
- This breaks the entry's "what stopped, and the way forward for each stop, one line each". It also breaks the summary's claim that "every error, warning and way_forward, at any depth, still prints on its own line … even when collapsed".
- An object list or deep object keeps its notes, but a plain list does not.
- Evidence: `list` returns right after the count line, and `notes` only runs from `item` and from the deep-object collapse in `field`. The `DeepObject` test covers `warnings` only inside a collapsed object, not a top-level long `warnings` list.
- Fix direction: in `list`, when `isNoteKey(key)`, skip the collapse (or print each string on its own line via `noteLines`).
- Plural `errors` is not recognised by `isNoteKey`. If any verb emits it, it needs adding.

**F2: LOW, design.** `internal/clihelp/readable.go`, `UseReadableOutput`, last two lines.
- `cmd.SetOut(summary)` runs unconditionally, even when `onFinish` registers nothing because the context carries no exit state. In that case a held unterminated line is never flushed.
- Production always has the exit state, so the impact is small.
- Fix direction: make `onFinish` report whether it registered, and only call `SetOut` when it did.

**F3: LOW, design.** `internal/output/summary.go`, `Summary`'s top-level loop (`if key == okKey || key == errorKey { continue }`).
- On `{"ok":true,"error":"…"}` the error string is dropped silently.
- No known producer emits this.
- Fix direction: skip the `error` key only when `ok` is false.

## Tool calls
No tool calls were made. My context already held every file I traced.
