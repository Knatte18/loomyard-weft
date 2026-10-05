# shedrecipe: capability declaration instead of manual seam-threading

`shedrecipe-capabilities` · Note · enhancement, shed

From `manifest/roadmap.md` Someday.
Giving a producer a new capability means hand-threading a passthrough `Env` field through three layers, because the Shed Recipe Registry entry (`PATTERN-shed-recipe-registry`) bars `shedrecipe` from importing the capability's owning package.
The idea, not yet designed: let a producer declare what it needs and have the registry wire it.
Deep, likely touching the invariant itself and every registry entry.