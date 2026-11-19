# Card 7 — sandbox-builder

**Create:**
- `internal/lymdirs#HardenSandboxDirName`
- `internal/fabricengine#SeedLocalBares`
- `internal/fabricengine_test#TestSeedLocalBares`
- `internal/hardenshed#SandboxRoot`
- `internal/hardenshed#SandboxDeps`
- `internal/hardenshed#SandboxHub`
- `internal/hardenshed#NewSandbox`
- `internal/hardenshed#ListSandbox`
- `internal/hardenshed#TestNewSandbox`

**Extends:**
- `internal/lymdirs/doc.go`
- `internal/fabricengine/doc.go`
- `internal/hardenshed/doc.go`

**Intent:**
The throwaway sandbox hub builder behind `lym harden sandbox new` and `list` (D5).

`lymdirs.HardenSandboxDirName` names the sandbox root's directory under the run worktree's never-tracked `.lym`; `SandboxRoot` joins the told worktree root with `.lym` and it, so every process of a run derives one root, and the root goes with the worktree.

`fabricengine.SeedLocalBares` creates, under `dir`, a bare warp holding one commit with a root README on `main`, HEAD re-pointed to `main`, and an empty bare weft, through `gitexec`, the shape `fabriccli.CloneAndWire` clones from; it is production code of fabric's own, and `internal/hubforge` stays test-only.
`TestSeedLocalBares` is tier1: the two bares exist, the warp's `main` holds the README commit, the weft has no ref, and `fabricengine.CloneHub` clones a hub from them.

`SandboxDeps` carries the root, `Seed func(dir string) (codeBare, recordsBare string, err error)` (wired to `SeedLocalBares`), `Clone func(codeBare, recordsBare, container string) (hubPath string, err error)` (wired by card 16 to `fabriccli.CloneAndWire` with no GitHub remote and no wiki), `CommitOverride func(hubPath string) error`, the worktree's `.dev-bin` lym path, `Head func() (string, error)`, `Revision func(path string) (buildvcs.Identity, error)` and `Deploy func() error`.
`hardenshed` lies outside PATTERN-fabric-boundary's owner set, so its doc and code name the two bares by role, the code bare and the records bare, and never by side name; `fabricengine.SeedLocalBares`, inside the owner set, keeps its signature.
`NewSandbox`, writing only under the root:
1. Makes a fresh hub container under the root, seeds the bares there, and clones the hub.
2. Calls `CommitOverride`, which card 16 wires to commit a `loom.yaml` with `selfreport: false` and `friction: ""` onto the hub's prime records before any pair exists.
3. Records the hub path in the root's hub list, `hubs.json`, written under `internal/state`'s lock, so another process lists it.
4. Calls `Deploy` only when the `.dev-bin` lym's revision is not `Head`'s or is marked modified.
5. Returns `SandboxHub{Hub, Lym}`, the hub's path and the absolute `.dev-bin` lym path.
A failure at any step after the container is made removes the container, the seeded bares included, before the error is returned, so no unrecorded `.git` is left under the root.
`ListSandbox` reads the hub list alone; a hub's entry outlives its directory until a reap prunes it (card 8).

`TestNewSandbox` is tier0 with fake seams: each call makes a hub under the root and writes nothing outside it; the hub list holds every hub made, read back by `ListSandbox` with no state held from the call; `Deploy` runs when the revision differs or is modified and not when it is `Head`'s; a failing `Clone` records no hub and leaves no container under the root.

**ImpactSummary:** One lymdirs name, one fabric function and new hardenshed code; no existing caller changes.
