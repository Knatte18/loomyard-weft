---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-forkaudit-card-arm.md (extract.jq and report.jq bullets)
    summary: >-
      The card arm's span-reached window differs from every other arm's, yet both share the same Pack use rows and labels, and the only disclosure is a jq source comment
    fix: >-
      In 04-forkaudit-card-arm.md, extend the report.jq bullet so that, whenever the card arm is present, report.jq prints one note line directly under the Pack use table:
      "Card arm: a span counts as reached when read after the card-pack call that lists it and before the next edit; the other arms count a span read before the batch's first edit, and the card arm has no first-work-turn share because that turn reads card files only."
      Keep the header-comment sentence as well.
      In the extract.jq bullet, state the rule for a span two calls list identically: it counts as reached when some call listing it precedes a read of it that precedes the next edit after that call.
      In the Tests section add to the existing fork-card row (trace card, one card pack line) a wantReport entry holding "Card arm: a span counts as reached when read after the card-pack call that lists it",
      and add to the `--runs alpha` row an assertion that the report holds no "Card arm:" line.
      The fix changes card text only, so no validate-plan check constrains it.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/plan/04-forkaudit-card-arm.md (extract.jq bullet; tools/forkaudit/extract.jq ex_net and pre_edit_net)
    summary: >-
      The net exploration and net calls-before-edit columns net out the file arm's pack read but not the card arm's card-pack calls, so the card arm reads one call per card worse
    fix: >-
      In the extract.jq bullet add: for a fork of the card arm, ex_net and pre_edit_net exclude the card-pack calls counted in card_pack_calls that fall inside the counted range, as the file arm's exclude its pack read; ex_tools, tools and the other raw columns keep them.
      In the Tests section give the fork-card row (trace card, one card pack line) the cells `calls before edit` 3, `net before edit` 2 and `net exploration` 2:
      the fork's tool calls before its first Edit are the refused card-pack call, the successful card-pack call and the span Read, and only the successful call is netted.
      The fix changes card text only, so no validate-plan check constrains it.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-forkaudit-card-arm.md (pack-entries.jq bullet)
    summary: >-
      Pack-check would class spans from card-pack calls for card 2 and later against the batch's start sha, although each is resolved after the earlier cards' edits, so valid spans show as stale, wrong or nofile
    fix: >-
      In the pack-entries.jq bullet replace "so --pack-check classes the per-card spans like any other" with: for a fork of the card arm the entries come from its first successful card-pack call only, the one resolved against the batch's start tree;
      later calls are resolved against trees the start sha does not hold, so their entries are left out of pack-entries.tsv, and the header comments of forkaudit.sh and report.jq say the card arm's pack check covers its first call.
      Pin it in the fork-card.jsonl bullet by adding, after the Edit, a successful `lyx loom card-pack 2` call whose result lists one entry `pkg#Stale` spanning `pkg/a.go:12-14` with signature `func Stale()`;
      then TestForkauditPackCheck wants agent-fcard `correct` 1 and `stale` 0, and TestForkauditTraceCrossCheck's agreeing fork-card row wants `spans` 2 with `strict before edit` still 1, since nothing reads the second span.
      The fix changes card text only, so no validate-plan check constrains it.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/03-per-card-layout.md (Edit list)
    summary: >-
      Four fragment banners still name kickstart_pack_file as the sole layout condition and become false once kickstart_pack_per_card takes precedence, but the card does not list them
    fix: >-
      Add these four lines to the Edit list of 03-per-card-layout.md, after the `webster-fragment-pack-read.md` line:
      `contracts/stencils/webster/webster-fragment-locate-pack.md`,
      `contracts/stencils/webster/webster-fragment-fresh-read-pack-inline.md`,
      `contracts/stencils/webster/webster-fragment-locate-pack-inline.md` and
      `contracts/stencils/webster/webster-fragment-pack-section-inline.md`.
      Add one sentence to the Intent next to the banner sentence for the two listed fragments:
      these banners read "kickstart_pack_per_card is false and kickstart_pack_file is true" for the file layout and "kickstart_pack_per_card is false and kickstart_pack_file is false" for the inline layout, and the text after each banner stays byte-identical.
      Checked with `lyx loom validate-plan --overlay` over the edited card against the `{}` baseline: ok true, no new finding.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/02-card-pack-verb.md (Intent, the paragraph on loom's pre-run)
    summary: >-
      The card says a card-pack call outside a hub refuses at loom's pre-run, but the pre-run passes in any git repository at its anchored directory and the verb itself refuses on the missing plan directory
    fix: >-
      In 02-card-pack-verb.md replace the sentence "A `lyx loom card-pack` call outside a hub worktree refuses at loom's pre-run, so a webster run in standalone geometry never gets the per-card prompt: card 3's begin-batch renders the file or inline layout there." with:
      "Loom's pre-run only needs a git repository at its anchored directory, so a call from a standalone target passes it, and the verb refuses on the missing plan directory, since a standalone run's plan lives in its state directory and not under the anchor; its way forward is the locate fallback, so a standalone webster run never gets a working pack from the verb and card 3's begin-batch renders the file or inline layout there."
      The existing test row for an anchor with no `_lyx/plan` directory already reaches this refusal.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/01-per-card-switch.md (Config doc comment and template.yaml comment)
    summary: >-
      The operator-facing doc and template comments for kickstart_pack_per_card say true picks the per-card layout, but a standalone run keeps the file or inline layout, which card 3 documents in doc.go only
    fix: >-
      In 01-per-card-switch.md add to the Config field's doc-comment description and to the template.yaml comment description:
      the per-card layout applies to a hub run, a standalone run keeps the layout kickstart_pack_file picks because lyx loom card-pack cannot run there, and every begin-batch reads the key afresh.
      Both targets, `internal/websterengine#Config` and `internal/websterengine/template.yaml`, are already on the card, so no target changes.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/04-forkaudit-card-arm.md (Tests, mainForks bullet, and the fork-card.jsonl bullet)
    summary: >-
      The card says every other row over mainForks takes the same count change, which is wrong for the runs-alpha and until rows, and it never fixes fork-card's first timestamp, which decides the until row
    fix: >-
      In 04-forkaudit-card-arm.md replace "and every other row over `mainForks` takes the same count change" with:
      "the `--runs alpha` row keeps its three kept forks because fork-card sits in run alpha-two, and the `until` row keeps its three kept forks because fork-card's default modification time is past its bound".
      In the fork-card.jsonl bullet add: its first line is stamped 2026-10-14T09:06:10.000Z, so its modification time two minutes later is past the until bound 2026-10-14T09:04:40Z, and its last line is at least 3 seconds after its first.
      The fix changes card text only, so no validate-plan check constrains it.
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/04-forkaudit-card-arm.md (report.jq bullet, the Card pack disagreements rule)
    summary: >-
      The disagreement rule lists a fork whose calls and traced count both exist and differ, which leaves a fork with no calls and a nonzero traced count undecided
    fix: >-
      In the report.jq bullet replace "A fork whose calls and traced count both exist and differ" with:
      "A fork with a card-traces.tsv row whose card-pack call count differs from the row's count, a count of 0 calls included".
---

### [MEDIUM] Card arm span window is not comparable to the other arms and is disclosed only in source

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, `extract.jq` and `report.jq` bullets; `tools/forkaudit/report.jq` Pack use rows.

**Issue:** For the file, raf and inline arms, `spans_strict_pre` and `spans_file_pre` count a span read before the batch's first edit (`tools/forkaudit/extract.jq`, `$pre` is every tool call before the transcript's first edit).
The card gives the card arm a per-card window: after the card-pack call that lists the span and before the next edit.
A card 2+ span of a file-layout fork is read late, after card 1's edits, and so counts as not reached, while the same behavior is reached by construction for a card-arm fork.
The Pack use rows "spans reached before the first edit, strict share", the file-level share rows and "forks reaching half their spans" carry one label for both windows, so the cross-arm comparison this task exists to enable reads as a like-for-like number.
The first-work-turn rows are made null for the card arm, which is right, but "forks with 3+ parallel calls in the first work turn, share" is left in place: it compares the card arm's turn of card-file reads with the file arm's turn of span reads.
The card's only disclosure is a header comment in `report.jq`, which the operator reading the report never sees.
It also leaves a span that two card-pack calls list identically (two cards sharing a `Uses:` member whose lines the first card did not move) without an owner, because `spans` is `unique`d on file and lines before the window is evaluated.
Resolving the focus question: the windows are not comparable as the plan stands, and the plan states the card arm's window only in source.

### [MEDIUM] Card-pack calls are not netted out of the net columns

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, `extract.jq` bullet; `tools/forkaudit/extract.jq` `ex_net` and `pre_edit_net`.

**Issue:** `ex_net` is `$ex.n` minus one when the fork opened a pack file before its first edit, and `pre_edit_net` is the first edit's index minus the same one.
Both count every tool call in an exploration segment, so a card-arm fork's `lyx loom card-pack` Bash calls, one per card, stay in both.
The file arm's single pack fetch is netted and the card arm's N fetches are not, which makes "exploration calls, net" and "calls before the first edit, net" read about one call per card higher for the card arm in the very comparison the report exists for.
The card adds `card_pack_calls` and says nothing about the net columns; whether the fetch calls are netted is a choice the author has to make and pin.
Netting them, as the fix does, keeps "net" meaning "beyond the pack fetch" in both layouts.

### [MEDIUM] Pack-check grades later cards' card-pack spans against the batch start

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, `pack-entries.jq` bullet; `tools/forkaudit/forkaudit.sh` `classify_span`.

**Issue:** `--pack-check` classes each span entry "against the file at the batch's start sha" (`classify_span`, and the report section "Pack correctness at batch start").
The per-card layout resolves card 2's pack against the tree after card 1's edits and commit, which is its purpose.
Grading that entry against the start sha gives `stale` for any span card 1 shifted, `nofile` for a file card 1 created and `wrong` for a member card 1 added, so correct packs count as defects, and the card arm looks less accurate for the reason the layout was built to improve.
The fixture covers one call for card 1, where the batch start and the card start are the same tree, so no row exposes it.
The plan's sentence "so `--pack-check` classes the per-card spans like any other" is the false premise.

### [LOW] Four fragment banners left naming kickstart_pack_file as the only condition

**Location:** `_lyx/plan/03-per-card-layout.md`, Edit list; `contracts/stencils/webster/webster-fragment-locate-pack.md`, `webster-fragment-fresh-read-pack-inline.md`, `webster-fragment-locate-pack-inline.md`, `webster-fragment-pack-section-inline.md`.

**Issue:** The locate-pack banner says it is read "in the file layout (kickstart_pack_file true)", and the three inline banners say they are read "when begin-batch built a kick-start pack and kickstart_pack_file is false".
Under the new precedence a true `kickstart_pack_per_card` reads neither, so each banner states a condition that no longer decides the fragment.
The card already updates the banners of `webster-fragment-card-read-pack` and `webster-fragment-pack-read` for the same reason, and the commit that introduced `kickstart_pack_file` (`da58615cd`) edited the locate-pack banner the same way.
Only the banner comments change; the inline fragments' rendered text stays the frozen text.

### [LOW] Wrong place named for a non-hub card-pack refusal

**Location:** `_lyx/plan/02-card-pack-verb.md`, Intent, the sentence ending "card 3's begin-batch renders the file or inline layout there".

**Issue:** `resolvePersistentPreRun` calls `lyxcwd.Resolve`, which fails only outside a git repository or off the anchored directory (`internal/lyxcwd/lyxcwd.go`, `buildLocation` has no hub check), and `card-pack` is not a generic shed verb, so the seed check does not apply.
A fork of a standalone run stands at the target's root, so the pre-run passes and the call reaches `CardPackMarkdown`, which refuses because the target holds no `_lyx/plan` (the standalone plan sits in the state directory, `internal/standalonegeom/webstergeom.go`).
The conclusion the card draws, and the fallback it relies on, hold; the mechanism it states does not, and the card 3 text and `doc.go` sentence will repeat whatever the card says.
Resolving the focus question: a standalone run gets the layout `kickstart_pack_file` picks (card 3's `packLayoutOf` and its standalone test row, which pins `pack_layout=file` and a written pack file), so it is stated and bounded.

### [LOW] Operator-facing comments for the new key omit the hub-only limit

**Location:** `_lyx/plan/01-per-card-switch.md`, the `Config` doc and `template.yaml` comment descriptions.

**Issue:** Card 1 tells the operator that `kickstart_pack_per_card: true` picks the per-card layout and overrides `kickstart_pack_file`.
Card 3 makes a standalone run ignore it, but states that in `doc.go` alone, so an operator reading `webster.yaml` in a standalone setup is told the key works there.
The behavior is bounded by card 3's test, and only the comment text that tells the operator about it is missing.

### [LOW] mainForks count sentence is wrong for two rows, and the fork-card timestamp is open

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, Tests, `mainForks` bullet.

**Issue:** The card places `fork-card` in run `alpha-two` and also says "every other row over `mainForks` takes the same count change".
The `--runs alpha` row keeps three kept forks because `alpha-two` is not selected, and the `until` row's three kept forks depend on the fixture's first timestamp, which the card never fixes: `forkaudit_integration_test.go` keeps a fork only when its modification time, by default its first line plus two minutes, is not after `2026-10-14T09:04:40Z`.
A first line earlier than 09:02:40 makes the `until` row keep four forks.
A literal reading breaks both rows.

### [NIT] Disagreement rule leaves the zero-call case undecided

**Location:** `_lyx/plan/04-forkaudit-card-arm.md`, `report.jq` bullet.

**Issue:** The column rule covers every combination of calls and traced count, but the disagreement rule lists a fork "whose calls and traced count both exist and differ", and a fork with no calls and a traced count of 2 prints `0/2` without a stated listing decision.
No Tests row reaches it, so the wording is all the implementer has.

## Rejected

- Focus 1, the `card packs` column: the rule gives every Tests row a value.
  A file-layout fork whose worktree has a trace file gets a `card-traces.tsv` row with count 0 and no call, which the first clause maps to `-`, and the `1/1`, `1/0` and `1/-` rows each fall under one clause.
  Only the zero-call disagreement wording (the NIT above) is open.
- Focus 4, the empty-card tests: the `go` plan with a plain path and a `plan:` handle and the `language: none` plan each have a named row, and each reaches its branch (`renderCardPack` prints the empty-card line when no member glyph is resolved or unresolved, and `kickstartPack` returns nil for a glyph-less language).
  The destination comments of the three pack fragments and `WebsterFragmentPackCard` are covered by card 2; the four banners of the finding above are a different set.
- Splitting `CardPackMarkdown` from the verb into its own card: bundled, since a separate card needs its own test of behavior the table test over the verb already covers (`PATTERN-test-economy`) and a budget row raise for `internal/websterengine`.
- Using `AnchorRoot == WorktreeRoot` as the hub discriminator in `packLayoutOf`: `internal/websterengine/geometry.go` documents the standalone pair as deliberately disjoint and the hub pair as equal, and the card pins both with tests.
- Test budget: card 2 adds one top-level test to `internal/loomcli`, which has headroom, and the other cards add rows only, so no budget row changes.
- Round 1: its five findings are all fixed in the plan and none regressed, and its fixer report disputes and defers nothing.
  The round-1 review and the card both said a standalone call "refuses at loom's pre-run"; the finding on card 2 above corrects that mechanism and leaves the round-1 conclusion standing.
