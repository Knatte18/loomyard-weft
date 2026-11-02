# Operator verbs print a readable summary on a terminal

A verb the operator types, such as `lym adopt root`, used to print one long JSON envelope.
On a stop that envelope carried every commit hash and every run record, which hid what happened and what to do next.
Now, when a command carries the `operator` audience and its output is a terminal, each envelope line it writes prints as a short summary.
The global `--envelope` flag keeps the JSON.

## What the summary shows

`output.Summary` renders any envelope, so no verb needs its own renderer:

- the first line is `ok`, or `failed: ` and the error;
- the other fields follow in key order, and a field that is empty or false is left out;
- a list of plain values, such as a divergence's commit hashes, prints on one line, or as its count when it holds more than five;
- a list of objects prints one line per item: its label (`slug`, `subject`, `name`, `branch` or `id`) and its one-line field (`line`, `message`, `title`, `state`, `status` or `outcome`);
- an object nested more than two levels deep prints as its field count;
- every error, warning and `way_forward`, at any depth, still prints on its own line, named by its path, even when its object or item is collapsed;
- a summary that left anything out ends with `full envelope: rerun with --envelope`.

`lym board list`, for example, now prints one `<slug>: <title>` line per entry.

## Wiring

- `output.SummaryWriter` is a stream filter. It holds a line that starts with `{` until the line ends and prints it as its summary when the line holds an envelope. Every other byte passes straight through, so prompts, progress lines and a verb's own text views are unchanged.
- `clihelp.UseReadableOutput`, called from the root's pre-run in `cmd/lym/main.go`, switches a command's output to that writer. It does so only when four things hold: the command carries the operator audience, its output is a terminal, no envelope flag is set, and the command declares no `--json` flag of its own. A command with its own `--json` flag, such as `lym shed status`, already renders its own terminal view.
- `clihelp.InstallReadableOutput` declares the global `--envelope` flag. A command that declares its own `--envelope`, as `lym creel` does, shadows it, and its own flag still keeps the JSON when set.
- The per-invocation exit state now holds steps that run once execution ends. `RunRootCtx` runs them before it writes a cobra error envelope, and that is where a held line that never ended gets flushed.

## Design decisions

- **Who gets the summary is decided by the terminal.** The board entry left open whether agents call these verbs and so need the flag in their stencils and skills. They do: the orch runs operator verbs. But an agent's shell output is a pipe, never a terminal, so agents, scripts, detached children and tests all keep the envelope. No stencil, skill or role file changes. `lym shed status` already uses the same terminal rule.
- **One generic renderer, not one per verb.** The operator audience covers about 150 verbs, each with its own envelope shape. The renderer keys on field names already in common use (`error`, `*_warning(s)`, `way_forward`, `line`, `slug`), so every operator verb gets a summary now, and a verb that wants a better one shows it by naming its fields that way.
- **The flag is `--envelope`, not `--json`.** The global `--json` already means "print the help as JSON and run nothing". `lym creel` already uses `--envelope` with this meaning.
- **Object lists never collapse.** For a listing verb, one line per item is the useful output. Only plain-value lists and deep objects collapse.

## Tests

- `internal/output`: `TestSummary` pins the rendering rules and rejects lines that are not envelopes. `TestSummaryWriter` pins an envelope split across writes, pass-through text, and a line that never ends, held until `Flush`.
- `internal/clihelp`: `TestUseReadableOutput` pins when the summary prints: an operator verb on a terminal gets it. `--envelope`, the role audience, an output that is no terminal and a command with its own `--json` all keep the envelope. A line that never ended is flushed at the end of the invocation.
