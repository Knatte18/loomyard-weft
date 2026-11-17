# Card 5 — codex-developer-instructions

**Create:**
- `plan:internal/shuttleengine/codexengine#instructionItem` -> `type instructionItem struct`
- `plan:internal/shuttleengine/codexengine#developerInstructionsKey` -> `const developerInstructionsKey = "developer_instructions"`
- `plan:internal/shuttleengine/codexengine#maxArgumentBytes` -> `const maxArgumentBytes = 128 * 1024`
- `plan:internal/shuttleengine/codexengine#developerInstructions` -> `func developerInstructions(items []instructionItem) (string, error)`

**Edit:**
- `internal/shuttleengine/codexengine#Codex.Prepare`
- `internal/shuttleengine/codexengine#Codex.OpenLaunch`
- `internal/shuttleengine/codexengine#launchLine`
- `internal/shuttleengine/codexengine#codexCommand`
- `internal/shuttleengine/codexengine#TestBuildLaunchCmd`
- `internal/shuttleengine/codexengine#TestPrepare_Launch`
- `internal/shuttleengine/codexengine#TestPrepare_SandboxMode`
- `internal/shuttleengine/codexengine#TestPrepare_ExpandsHomeInWritableRoots`
- `internal/shuttleengine/codexengine#TestPrepare_InstallsGuardsFromItsInputs`
- `internal/shuttleengine/codexengine#TestPrepare_AppendsStandingPassages`
- `internal/shuttleengine/codexengine/guards.go`
- `internal/shuttleengine/codexengine/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/shuttleengine/codexengine#promptText`

**Uses:**
- `internal/shuttleengine/codexengine#tomlString`
- `internal/shuttleengine/codexengine#TestPrepare_SkillsLoadFirst`
- `internal/shuttleengine#StandingPassages`
- `internal/shuttleengine#CreelTrustSentence`

**Intent:**
A Codex launch passes its standing items as Codex's `developer_instructions`, and the run's prompt file holds only the spec's prompt (D5).

`instructionItem` is `{name, text, wayForward}`: `name` labels the item in a refusal, and `wayForward` is the step that shrinks it.
`Prepare` builds the items in the order Claude's appended system prompt file holds them:
the deny notice and `CreelTrustSentence` as one item, as `promptText` joins them today, then each standing passage as its own item.
Later cards add the subagent passage after the first item (card 6) and the user setup and instruction-file pointers after the passages (cards 9 and 10).
`prompt.md` is written with `spec.Prompt` alone, and `promptText`, whose one caller is `Prepare`, is deleted.

`developerInstructions` joins the non-empty items' texts with blank lines.
An empty set returns "", and `codexCommand` then omits the key, since Codex refuses a blank value.
Otherwise the setting is `developer_instructions=<tomlString(text)>`, a single-line TOML string, and `codexCommand` passes it as `-c <sh.Quote(setting)>` after the effort setting.
Every `-c` setting this plan adds sits before the hook settings and `--dangerously-bypass-hook-trust`,
so `TestPrepare_SkillsLoadFirst`'s assertion that the hook-trust flag directly precedes the first turn stays true.
Bound: that setting reaches `codex` as one argument, which Linux caps at `maxArgumentBytes` (`MAX_ARG_STRLEN`) counting its terminating NUL,
so the longest argument `execve` takes is `maxArgumentBytes`-1 bytes.
When the setting's length is at least `maxArgumentBytes`, `developerInstructions` refuses with an error naming the setting's size, the cap, the largest item's name and size, and that item's way forward.
A setting of `maxArgumentBytes`-1 bytes goes ahead.
`Prepare` returns it with the `codexengine:` prefix before `prompt.md` is written.
A standing passage's name is `heddle passage <n>`; its way forward is to find the entry with `lym heddle show codex-prompt` and shorten it with `lym heddle set`, or with `lym heddle set --layer lym` for a shipped entry.
Cards 9 and 10 name the file and plugin items and their ways forward.
The input that reaches the refusal after every upstream check passes is a standing passage over 128 KiB from the installed passage reader.

`OpenLaunch` carries no developer instructions: its line is unchanged.
Its doc comment's reason for carrying no creel hooks changes from "no prompt file to hold the creel trust sentence" to "no developer instructions to hold the creel trust sentence".
`guards.go`'s file comment says the rows' notice opens the developer instructions, not the prompt file.
`docs/overview.md`'s Codex paragraph says a Codex run carries the creel trust sentence in its developer instructions.

Tests:
- `TestBuildLaunchCmd`'s `EveryInput` row sets `developerInstructions` to text holding a quote and a newline, and its want carries the quoted single-line setting;
  the `NoWritableRootModelOrEffort` row keeps it empty and its want carries no key.
- `TestPrepare_Launch`, `TestPrepare_SandboxMode`, `TestPrepare_ExpandsHomeInWritableRoots` and `TestPrepare_InstallsGuardsFromItsInputs` build their want line with the deny notice and creel sentence as `developerInstructions`,
  and those that read `prompt.md` expect the spec's prompt alone.
- `TestPrepare_AppendsStandingPassages` (non-parallel, installs the passage reader) becomes a table.
  Its `PassagesFollowTheCreelHead` row expects the launch line's developer instructions to hold the creel head, then the two passages, and `prompt.md` to hold the prompt alone, keeping its reader-argument and trust-root assertions.
  Its `AtTheArgumentCap` row's reader returns one passage sized so the setting is `maxArgumentBytes`-1 bytes; the launch goes ahead.
  Its `OverTheArgumentCap` row's reader returns one passage one byte longer, so the setting is exactly `maxArgumentBytes` bytes;
  it expects a refusal naming the size, `heddle passage 1`, `lym heddle show codex-prompt` and `way forward`, and no `prompt.md`.
  The `//testtiming:keep` reason gains the cap refusal and its boundary.

`codexengine/doc.go`'s paragraphs on what the prompt file holds (the creel trust sentence, the deny notice, an opened session's missing prompt file) are rewritten:
the standing items ride `developer_instructions`, refused past 128 KiB, and `prompt.md` holds the prompt alone.

**ImpactSummary:** Prepare's prompt file and launch line change for every Codex run; `promptText` has one caller, deleted with it; the four full-line Prepare tests and the passage test update; `TestPrepare_SkillsLoadFirst` keeps passing by the setting order; OpenLaunch's line is unchanged.
