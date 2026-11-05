# Distill PATTERN.md into cluster entries with detail in background files

`PATTERN.md` sat at its headroom line, so every run that added or widened an entry stopped at Plan-Write until trims of other entries were recorded.
This change groups entries that guard one subject into one cluster entry with a short rule line, and moves the detail into the cluster's background file.
The overview drops from 15,541 to 7,532 bytes, about 46% of `MaxOverviewBytes`, which leaves room for several runs to add entries before the next trim.
`internal/pattern`'s cap and headroom values are unchanged.

## What changed

- **Overview.** 64 entries become 30 under the same section headings.
  Ten entries keep their ID and line as they were (`PATTERN-cwd-resolution`, `PATTERN-leaf-packages`, `PATTERN-refusal-way-forward`, `PATTERN-github-auth`, `PATTERN-tolerant-records`, `PATTERN-sole-parsers`, `PATTERN-shuttle-provider-seam`, `PATTERN-shell-guard-table`, `PATTERN-test-speed`, `PATTERN-test-isolation`).
  The rest are clusters.
  A cluster keeps an existing ID where that ID still names the subject (`PATTERN-cli-cobra`, `PATTERN-config-layers`, `PATTERN-gitrepo-client-boundary`, `PATTERN-owner-machine`, `PATTERN-shed-recipe-registry`, `PATTERN-shed-driver`, `PATTERN-gate-slot`, `PATTERN-stencil-ownership`, `PATTERN-heddle-instructions`, `PATTERN-daemon-wakeups`).
  Otherwise it gets a new ID: `PATTERN-code-conventions`, `PATTERN-lym-files`, `PATTERN-fabric-boundary`, `PATTERN-pair-remote`, `PATTERN-landing`, `PATTERN-dobby-runs`, `PATTERN-run-records`, `PATTERN-reed-panes`, `PATTERN-agent-addressing`, `PATTERN-child-processes`.
  A cluster carries `(test)` when any of its members did.
- **Background files.** Each cluster's file under `pattern/` opens with what the cluster guards, then has one section per member.
  A section starts with `Rule:` and the member's old overview rule text, copied verbatim, followed by that member's old background text with its headings bumped one level.
  The background files of folded members are deleted, so every file under `pattern/` is still named by exactly one entry.
  Every old rule text is present verbatim in the new overview or a background file.
- **Citations.** Every citation of a folded ID, in Go comments and messages, tests, specs, stencil tests, `docs/overview.md` and the background files, now names its cluster's ID.
  A few comments that named the old entry on its own now say which part of the cluster they mean, such as "PATTERN-stencil-ownership's parent-directive marker".
  A grep for any folded ID finds hits only under `docs/research`.

## Design decisions

- **Clusters reuse IDs where they fit.** Keeping the most-cited member's ID as the cluster ID keeps the citation rewrite down to 43 files outside `pattern/`.
  A new ID was chosen only where no member's ID names the whole subject.
- **The overview format is unchanged.** A cluster is an ordinary entry line, so `internal/pattern`'s checker, projection and directive rendering need no code change.

## Not changed here

- The Bolt checkout's board entries and its deployed spec copy (`_bolt/_lym/specs/loom/loom-plan-checks-spec.md`) still cite folded IDs.
  The spec copy follows `contracts/specs` on the next deploy.
  A run in flight that declares a PATTERN change against a folded ID, such as drift-rework's design, has to name the cluster ID instead.
