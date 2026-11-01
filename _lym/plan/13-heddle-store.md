# Card 13 — heddle-store

**Create:**
- `internal/heddleengine/doc.go`
- `plan:internal/heddleengine#FileName` -> `const FileName = "heddle.json"`
- `plan:internal/heddleengine#Target` -> `type Target struct`
- `plan:internal/heddleengine#targetTable` -> `var targetTable = []Target{}`
- `plan:internal/heddleengine#LookupTarget` -> `func LookupTarget(name string) (Target, bool)`
- `plan:internal/heddleengine#TargetNames` -> `func TargetNames() []string`
- `plan:internal/heddleengine#Entry` -> `type Entry struct`
- `plan:internal/heddleengine#Entry.MarshalJSON` -> `func (e Entry) MarshalJSON() ([]byte, error)`
- `plan:internal/heddleengine#Entry.UnmarshalJSON` -> `func (e *Entry) UnmarshalJSON(data []byte) error`
- `plan:internal/heddleengine#Store` -> `type Store struct`
- `plan:internal/heddleengine#Store.MarshalJSON` -> `func (s Store) MarshalJSON() ([]byte, error)`
- `plan:internal/heddleengine#Store.UnmarshalJSON` -> `func (s *Store) UnmarshalJSON(data []byte) error`
- `plan:internal/heddleengine#Decode` -> `func Decode(data []byte) (Store, error)`
- `plan:internal/heddleengine#Load` -> `func Load(boltDir string) (Store, error)`
- `plan:internal/heddleengine#Save` -> `func Save(boltDir string, s Store) error`
- `plan:internal/heddleengine#Store.Ordered` -> `func (s Store) Ordered() []Entry`
- `plan:internal/heddleengine#Store.Get` -> `func (s Store) Get(slug string) (Entry, bool)`
- `plan:internal/heddleengine#Patch` -> `type Patch struct`
- `plan:internal/heddleengine#Store.Set` -> `func (s *Store) Set(slug string, p Patch) error`
- `plan:internal/heddleengine#Store.Remove` -> `func (s *Store) Remove(slug string) error`
- `plan:internal/heddleengine#ValidateEntry` -> `func ValidateEntry(e Entry) error`
- `plan:internal/heddleengine#TestStore` -> `func TestStore(t *testing.T)`

**Intent:** Add heddle's store, `heddle.json` at the Bolt checkout's root (D1), in a new `internal/heddleengine` package.

Every file of the package, tests included, spells no fabric side token in an identifier, string or comment (PATTERN-fabric-vocabulary bars it outside fabric's owner set): it says "the worktree's root", never the side name.
`doc.go` states the package's role (PATTERN-heddle-instructions): the store, the target table, the file and prompt rendering of cards 14 to 16, heddle's Bolt replayer; that it runs no git and is handed every path; and that lym's own appended sentences stay in `claudeengine`.

- The target table is heddle's code, one row per target, each `Target` naming `Name`, `Delivery` (`file` or `prompt`), `File` for a file target and `ReRender`, the root-relative paths whose change re-renders it:
  `claude-file` → `CLAUDE.local.md`, re-rendered on `CLAUDE.local.md`, `AGENTS.md`, `CLAUDE.md`, `.claude/AGENTS.md` and `.claude/CLAUDE.md`;
  `codex-file` → `AGENTS.override.md`, re-rendered on `AGENTS.md` and `AGENTS.override.md`;
  `claude-prompt` and `codex-prompt`, prompt deliveries with no file.
  `LookupTarget` and `TargetNames` read it; a further target is one new row.
- `Entry` holds `Slug`, `Text`, `Order` (an int) and `Targets`, plus `Unknown map[string]json.RawMessage` for the keys it does not know; `Store` holds `Entries` and its own top-level `Unknown`.
  Their `UnmarshalJSON` decodes the known keys and keeps every other key raw, and `MarshalJSON` writes the known keys then the unknown ones in key order, the way `boardengine`'s `storeFile` and `Task` do (PATTERN-tolerant-records); every rewrite, a replay's included, goes through these types.
- `Decode` decodes bytes, empty or whitespace bytes being an empty store; bytes that do not decode are refused naming `heddle.json` and the way forward, restoring the last committed file in the Bolt checkout with `git -C <Bolt checkout> checkout HEAD -- heddle.json` (or an earlier commit from `git -C <Bolt checkout> log -- heddle.json`), since every `lym heddle` verb and render reads the store first and so cannot repair it; `Load` reads `<boltDir>/heddle.json`, an absent file holding no entry; `Save` writes it atomically with `fsx.AtomicWrite`, indented, ending in a newline.
  Neither takes a lock: the callers do (cards 14 and 18).
- `Ordered` returns the entries by ascending `Order`, ties broken by slug.
- `ValidateEntry` refuses a slug not matching `[a-z0-9-]+`, an empty or whitespace-only text, no target, and a target the table does not hold, each message naming the way forward: the slug rule, a non-empty `--text-file`, or the valid target names from `TargetNames`.
- `Patch` carries an optional text, order and target list (nil means unchanged).
  `Set` updates an existing entry with only the fields given, or creates one, which needs all three and is refused naming the missing flags; the result passes `ValidateEntry` or nothing changes.
  `Remove` deletes an entry; an unknown slug is refused naming `lym heddle list`; `Get` reports whether a slug is held.
  Neither touches another entry's fields or unknown keys.

`TestStore` (tier0, in a new `internal/heddleengine/store_test.go` with `//go:build tier0`, temp directories only) rows:
a store holding an unknown top-level key and an unknown key on each of three entries keeps them all after `Set` updates the middle entry's text, after `Set` adds a fourth entry and after `Remove` drops the first, each saved and loaded again;
an absent `heddle.json` loads as no entry and `Ordered` returns none;
`Ordered` sorts by order and breaks a tie by slug;
`Set` refuses a bad slug, an empty text, an unknown target and a new entry missing its order or targets, each message naming its way forward and the store left as it was;
`Remove` of an unknown slug refuses naming `lym heddle list`, and `Get` of one reports it unheld;
a partial `Set` of an existing entry changes only the given field;
a `heddle.json` holding malformed JSON fails `Load` naming the file and the `git -C … checkout HEAD -- heddle.json` way forward.
