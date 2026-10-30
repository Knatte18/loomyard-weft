# Decision record: a closed area for lym so bypass permissions is safe

## Goal

Write one research report, `docs/research/sandboxing-options-2026-10-30.md`, that weighs ways to run the whole lym process tree in a closed area on the operator's Linux machines, with a short list of controlled exits, and recommends which to build first.

## Scope

- In scope: the six methods the board entry names, and only those: `bwrap`; Landlock through a wrapper; a dedicated Linux user; systemd sandboxing properties on a user scope; a rootless container (podman); Claude Code's own `/sandbox` with its permission and managed-settings features.
- In scope: Ubuntu 24.04 (OSL-1046) and Ubuntu 26.04 (Neve, Kubuntu, Plasma 6), their AppArmor and user-namespace rules and kernel versions.
- In scope: how reed's tmux server, the operator's `lym reed attach`, dobby, creel's daemon and sockets, and shuttle's agent sessions start inside each method, and what stops working.
- In scope: each exit the board entry lists, the smallest controlled way to give it and which methods can.
- In scope: the lym packages each method changes and a work estimate as a count of tasks or cards.
- In scope: what the repository can state about safe sandboxing, and what only for the method it ships.
- Out of scope: Windows, adopting any sandbox, changing lym code, and changing any setting of this machine.
- Out of scope: every hub but this task's worktree; the Models hub stays untouched.

## Decisions

### D1 — One report file, no code change

The change adds `docs/research/sandboxing-options-2026-10-30.md` and nothing else.
Rationale: the board entry asks for research and forbids adopting a sandbox or changing lym code.

### D2 — Methods weighed by name, in a fixed set of six

The report has one section per method the board entry names, and weighs no seventh.
Anthropic's `sandbox-runtime` (`srt`) and a systemd system unit are named only as implementation variants inside the `bwrap` and systemd sections.
Rationale: the board caps the methods at six by name; the two variants change the cost of a named method, not the method.

### D3 — Short probes on this machine are evidence

The report records what short probes of `bwrap`, `unshare` and `systemd-run --user` showed on OSL-1046, and says which ran under the `vscode` AppArmor label this session inherits and which ran from an unconfined systemd user service.
Probes create nothing outside `.scratch/` and leave no file behind.
`landrun` and `podman` are not installed here, so their sections rest on their documentation alone and say so.
Rationale: the board allows short probes and forbids changing the machine; a probe whose AppArmor label differs from a fresh login shell misleads unless the label is stated.

### D4 — Exits judged against a stated threat model

The report states its threat model first: an agent session that runs hostile commands, through a prompt injection or a bad change, and must not act outside the area, read secrets outside it or persist code that later runs outside it.
Each exit is judged by the smallest grant that keeps that property.
Rationale: "safe" has no meaning without the attacker it is safe against, and the exits differ only under one.

### D5 — Work estimates in tasks and cards

Each method's lym changes are named by package, and its work is counted in board tasks and Webster cards, never in hours.
Rationale: the board entry asks for it.

### D6 — One recommendation with an order

The report ends with the method to build first, the order of the rest and what to leave out, each with its reason.
Rationale: the board entry asks for a recommendation, not a survey.

## Constraints

- The report follows `scribe:prose`: one sentence per line, no padding, table cells on one line.
- Every claim about lym cites the package or file it rests on; every claim about a tool cites the documentation read, with its URL.
- No setting of this machine changes, no package is installed and no file outside this worktree is written.
- No lym code changes, so `PATTERN.md` and package docs stay as they are.

## Auto-mode assumptions

- A darn run holds no interview; the board entry is the whole statement of the change.
- The report path is the one the board entry names, dated 2026-10-30.
- Neve's facts (Ubuntu 26.04) come from documentation, since the run cannot probe that machine.

## Open risks

- Ubuntu 26.04 packaging of the `bwrap-userns-restrict` profile and of `podman` and `landrun` is documented only by secondary sources; the report marks each such fact as unverified on Neve.
- Landlock's pathname Unix-socket scope (ABI 9) and UDP rules (ABI 10) are newer than the kernel 7.0 both machines run, by the `landrun` README's table; a later kernel changes that method's verdict.
- Claude Code's sandbox and permission features change quickly; the report cites the version installed here (2.1.314) and the documentation as read on 2026-10-30.

## Acceptance criteria

- `docs/research/sandboxing-options-2026-10-30.md` exists and is the only file the change adds or edits (D1).
- The report has one section per named method, six in all, and the two variants appear only inside their parent method's section (D2).
- Each probe result states the AppArmor label it ran under, and the `landrun` and `podman` sections say they rest on documentation alone (D3).
- The report states its threat model before it judges any method or exit (D4).
- Each method names the lym packages it changes and counts its work in tasks or cards (D5).
- The report ends with a recommendation naming the first method, the order of the rest and what to leave out (D6).
- The report answers each of the board entry's six questions, and covers each exit the board entry lists: network, the GitHub token and push, the Claude config directory and its credentials, the landing deploy, the desktop windows and VS Code, and host git over a checked-out repository.
