# Card 11 — Shuttle's Start requirements refusal takes a caller's model key

**Create:**
- `plan:internal/shuttleengine#ModelKey` -> `type ModelKey struct`
- `plan:internal/shuttleengine#ModelKey.Wrap` -> `func (k ModelKey) Wrap(err error) error`
- `plan:internal/shuttleengine#startRequirementsError` -> `type startRequirementsError struct`

**Edit:**
- `internal/shuttleengine#Runner.checkRequirementsAtStart`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestRunner_Requires`

**Intent:**
D8's caller-wrapped refusal, the shuttle mechanism the seat card and the Master and driver card use.

Shuttle's `Start` refusal of an unmet requirement stays an error wrapping `ErrRequirementsUnmet` that names the engine, each unmet requirement and the engines that meet them all, and carries a generic way forward, so a refusal no caller wraps still carries one.
The caller it serves is `lyx orch start`: `orchSpec` sets `ForkSubagents`, passes `orch.yaml`'s model string verbatim and sets no `Engine`, so the orch has no alias to give and runs on `shuttle.yaml`'s default.
`checkRequirementsAtStart` returns it as a new unexported `startRequirementsError` holding the refusal without its way forward; its `Error` appends the generic clause `give the role a model alias of an engine that meets them, or for a role that takes no alias set engine: in shuttle.yaml to one that does, then re-run the verb`, and it unwraps to the held refusal, so `errors.Is(err, ErrRequirementsUnmet)` still holds.
The attach refusal and the unknown-requirement refusal keep their text.
`ModelKey{Module, Key, Entry, DefaultEngine}` names where a role's model came from; `Wrap` returns any error whose chain holds no `startRequirementsError` unchanged, and for one that does it returns the held refusal with the key-specific way forward in place of the generic one, still wrapping `ErrRequirementsUnmet`, in one of four forms:
- a key with `Module` and `Key` set and `Entry` empty, for a scalar model spec: `; way forward: set <Key> in <Module>.yaml to an alias whose engine meets them with lyx config <Module> --set <Key>=<alias>, then re-run the verb, or under the role's row re-step the row`;
- a key with `Entry` also set, for one entry of a `{name, model}` list: `; way forward: set the model of <Entry> in <Key> in <Module>.yaml to an alias whose engine meets them with lyx config <Module>, then re-run the verb, or under the role's row re-step the row`;
- `DefaultEngine` set, for a model that named no engine and resolved no key: `; way forward: set engine: in shuttle.yaml to an engine that meets them, then re-run the verb`;
- the zero key, for a seat whose key no caller names, as a MultiLLM recipe seat: `; way forward: set the seat's model to an alias whose engine meets them, then re-run the verb, or under the role's row re-step the row`.
So no requirements refusal leaves a seat or any other `Start` caller without a way forward.
`doc.go` states the split.

Tests:
- `TestRunner_Requires` asserts a refused `Start` names the engine and requirements and, unwrapped, carries the generic way forward with both halves, the model alias and the `engine:` in `shuttle.yaml` for a role that takes no alias, and adds `ModelKey.Wrap` rows over that refused `Start`'s error: under a named key it carries only the way forward naming `lyx config loom --set discussion=<alias>`, under a key with `Entry` `skeptic` in `discussion_advisors` only the one naming `the model of skeptic in discussion_advisors`, under `DefaultEngine` only the `engine:` one and under the zero key only the seat one, each still wrapping `ErrRequirementsUnmet` and none containing the generic clause; an unknown-requirement refusal and an error not wrapping `ErrRequirementsUnmet` pass unchanged under each.

**ImpactSummary:** shuttle's Start refusal becomes a typed error whose generic way forward a caller's `ModelKey` replaces; no caller wraps it yet, so every refusal still reads with the generic way forward.

**Commit:** 11: shuttle's Start requirements refusal takes a caller's model key
