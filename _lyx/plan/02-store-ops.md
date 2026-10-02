# Card 2 — store-ops

**Edit:**
- `internal/boardengine/store.go`
- `internal/boardengine/store_test.go`

**Intent:**
Add the three in-memory operations the new verbs need, on `Store`, independent of which file backs it (Decisions 14–16).

- `Promote(idOrSlug any, target *int) (Task, error)`: a nil target means the entry's tier minus one.
  The target must be at least 1 and strictly lower than the current tier; promoting a tier-1 entry, or to an equal or higher tier, is refused, and the message names the entry's current tier and says demotion goes through `upsert` with a `tier`.
  Skipping tiers is allowed.
  The promoted entry passes `validateWrite`, so a dependency left at a higher tier refuses the promotion, naming that dependency.
  A missing entry is refused as not found.
- `Prune() []string`: removes every entry whose status is `done`, strips the removed slugs from every remaining entry's `depends_on`, and returns the removed slugs in store order.
  An `abandoned` entry survives it.
  Design-doc cleanup needs no code here: the render sidecar drops the files of entries no longer rendered.
- `Find(text string) []BriefTask`: one case-insensitive substring matched against slug, title, brief and body of every entry, `done` included, returning the matching entries in the shape and order `ListTasksBrief` uses.
  An empty text is the caller's refusal, not the store's.

Tests in `store_test.go`: promote defaults to one tier lower, accepts a skip (3 → 1), and refuses an equal, a higher, a below-1 target and a tier-1 entry;
promote refused when a dependency sits at a higher tier than the target;
prune removes only `done` entries, keeps an `abandoned` one, strips the removed slugs from survivors' `depends_on`, and reports exactly the removed slugs;
find matches each of the four fields case-insensitively, returns a `done` entry, and returns an empty slice on no match.

**ImpactSummary:** Additive methods on `Store`; no existing caller changes.
