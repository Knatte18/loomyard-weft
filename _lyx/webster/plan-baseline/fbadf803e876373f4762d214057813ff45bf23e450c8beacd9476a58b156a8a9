# Card 1 — tokencount per-fork report

**Edit:**
- `tools/tokencount/count.go`
- `tools/tokencount/main.go`
- `tools/tokencount/count_test.go`

**Intent:**
`tools/tokencount` reports each Webster fork's token use and ties it to the cards it ran (Decision 2).

Every sub-agent transcript under a session whose role is `webster` (Master) is one fork.
`CountRun` builds one `ForkTally` per such transcript in the same pass that tallies roles, so the run-wide message-id dedupe applies unchanged: a message the fork repeats from Master's context is not counted again.
`RunTally` gains `Forks []ForkTally`.
A `ForkTally` carries:

- the run slug and the transcript's file name;
- `Cards`, the card ids (`NN-<slug>`) the fork ran, empty when unattributed;
- `Messages`, the assistant messages counted for the fork;
- `PeakContext`, the largest `input + cache write + cache read` of any counted message;
- `Usage`, whose `Weight()` is the existing weight figure;
- `Started`, the timestamp of the transcript's first line that carries one (the `timestamp` field of a transcript line).

Attribution reads the fork's prompt, never commit times.
A fork reads its prompt file with the `Read` tool, so its prompt is the first tool result in its transcript that holds a card pointer line.
A `Read` result prefixes every line with its number and a tab (`58<TAB>- ...`); the prefix is ignored when matching.
A card pointer line is the shape `renderCardPointers` in `internal/websterengine/render.go` renders: a bullet `` - `<path>/NN-<slug>.md` `` with nothing after the closing backtick, whose file name starts with two digits and a hyphen and is not `00-overview.md`.
The prompt's card gate lines hold the same backticked path followed by `: <command>`, so the nothing-after rule keeps them out.
The fork's cards are every such pointer line of that one tool result, in order.
A tool result's content is either a string or a list of text items; read both.
A fork whose transcript holds no such tool result is listed as unattributed, never dropped.

`Report.WriteMarkdown` appends, after the per-run role tables, a `## Webster forks` section: one table over every run's forks, ordered by run as listed and then by `Started`, with the columns run, cards (comma-joined ids, or `unattributed`), messages, peak context, weight.

The package comment in `main.go` describes the fork table, what one fork is and the attribution rule.

`count_test.go`: extend `TestCountRun`'s fixture so the `webster` session has two sub-agent transcripts, one whose first pointer-holding tool result, built in the `Read` form with a line-number prefix, names `01-alpha` and `02-beta` (plus a gate line naming a third card, and a later tool result naming another card file, neither of which may count) and one with no pointer, and assert both `ForkTally` values field by field (cards, messages after the dedupe, peak context, usage, started) and the two report rows exactly, the second reading `unattributed`.
`tools/` test files fall under the fabric-vocabulary scan: never name either fabric side in them.

**ImpactSummary:** `tools/tokencount` only; the role tables and their totals are unchanged, the report gains one section.
