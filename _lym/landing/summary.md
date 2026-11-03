# Darn-Write: writer runs no gate when its verify gate has a cold fixer

The Darn-Write writer ran `lym gate test` and fixed failures in its own session, at full context, though the cold gate fixer clears a failed `verify` gate after the writer hands off.
Plan-Write and Discussion-Write already switch to a "gated" self-check when their row's gate has a fixer; Darn-Write's Step 4 was left out.

Now, when `darn.gate_fixer` holds a model-spec and the worktree has a pair slug, the writer's prompt tells the writer and its subagents to run no `lym gate test`.
The row's `verify` gate and its fixer take the failures, and the writer acts only on what the fixer leaves.
With the fixer off, or with no pair slug, the Step 4 text is unchanged.

## Design

- Step 4's list moves out of `darn-template-write.md` into two fragments, `darn-fragment-make-change` and `darn-fragment-make-change-gated`, registered in `contracts/stencils`.
  The template holds one `{{.make_change}}` marker where the list was.
- `darnengine.DarnTable` reads the fragment its new `DarnInputs.GateFixed` field picks, fills the fragment's `subagent_model` marker, and fills the result into `make_change`.
  `subagent_model` is no longer a marker of the template itself.
  A missing fragment is refused by name.
- `loomcli.wireDarn` takes the worktree's pair slug and computes `GateFixed` through `writerFixerOn`, the same condition Plan-Write and Discussion-Write use and the same one `gateFixerSeam` uses to attach a fixer to the row's gate.
- The gated fragment differs from the ungated one in three places: no gate run per commit, a step saying the writer runs no gate and hears only what the fixer leaves, and subagent prompts that say to run no `lym gate test`.

## Tests

- `darnengine`: `TestDarnSpec` pins both renderings (gate step and subagent gate rule present or absent), and `TestDarnSpec_Refusals` covers each missing fragment.
- `loomcli`: `TestWire_DarnRecipe` checks the choice through `wire` for a pair slug with a fixer, a pair slug with an empty fixer, and a fixer with no pair slug.
- `contracts/stencils`: the marker tests cover both fragments and the `make_change` position.

## To check by hand

- `wireDarn`'s choice between the gated and ungated fragment, in `internal/loomcli/darnwiring.go`: confirm the wiring test in `wiring_darn_test.go` pins it, since a review left this finding (`wiredarn-gatefixed-choice-untested`, design, medium) open.
- A real Darn run with `darn.gate_fixer` set: confirm the writer runs no gate and the fixer's leftovers reach it.
  The tests above render the prompt only.
