---
round: 1
focus:
  - "internal/loomengine/config_test.go, TestLoadConfig_Loads: do the cases now derive `want` from the template's own load (`templateLoad`) so that a key added to loom.yaml needs no table edit, and is any case still pinning a whole struct?"
  - "internal/loomengine/config_test.go: a case that mutates a field to the value the template already loads to would pass vacuously; does any case's mutate (or the Perch/soft-bounce expectations) lose the assertion it had when compared against a hard-coded whole Config?"
  - "internal/loomengine/config_test.go, `template defaults` subtest: does the reflect/FieldByName check cover every field the old templateConfig() pinned, with equal values, and does it fail loudly on a renamed field rather than silently skipping?"
  - "internal/configreg/configreg_test.go, TestModules_EveryTemplateKeyClassified: are the new failure messages correct about which map or list to edit (the Categories map of the Spec function, wantLocalKeys, wantLocalDenied, wantMachineKeys), and is `runtime.FuncForPC(reflect.ValueOf(m.Spec)...)` sound for every module's Spec (method values, closures, nil)?"
  - "internal/configreg/configreg_test.go, keyListWayForward and keysOutside: do add/drop directions match got vs want (the test's expected list gains what got holds and want lacks), and is the order-only mismatch case handled?"
  - "Decision record Idea, third sentence: the diff touches only the two test files; does it check and list the other whole-struct or whole-key-list tests (TestLookup, TestWire_ReviewKeysReachTheEnv, TestRecipes_KeySet) as the Idea asks, or leave them, and does _lym/landing/summary.md claim anything the diff does not bear out about them?"
  - "Test economy: does the new `template defaults` subtest or any helper duplicate coverage an existing test already gives, and is any new behavior (message helpers) left without a test?"
---

The measuring stick is the Goal entry in the decision record alone; its other sections are empty.
The diff covers two test files and carries no production code.
