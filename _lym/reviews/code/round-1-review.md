---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/creelengine/crosshub.go:36
    summary: "widestStampedLine omits SendingRepository, so the pre-post length check undercounts the stamp that now leads with owner/repo"
    fix: In internal/creelengine/crosshub.go set SendingRepository on the Message built in widestStampedLine to the longest legal GitHub name (39-character owner, slash, 100-character repo), and add a row to the existing validateStampedLine / over-budget test in internal/creelengine proving a body at the limit still fits once the repository leads the stamp.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/creelengine/journal_test.go:802
    summary: "SendingRepository is journaled, replayed (applyInbound) and carried (applyCarried, carryRecords) with no test that pins it through replay or carry"
    fix: In the existing journal replay and retention-carry tests in internal/creelengine/journal_test.go (the rows near lines 261 and 802-840), set SendingRepository on the inbound record and assert the folded Message still carries it after replay and after a carry across the retention boundary.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/creelcli/cli.go:326
    summary: "the sending_repository envelope field is emitted when set and omitted when empty, and no test asserts either"
    fix: In the existing cross-hub read test in internal/creelcli/cli_test.go assert sending_repository equals the repository for a message from a hub with one, and is absent for a message from a hub with none.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configreg/configreg.go:164
    summary: "the creel module's reinterpretation registration, repositoryOfShortname and hubregistry.LookupRepository have no test, so dropping the Reinterpretations entry passes every test"
    fix: Add a row to the configreg module-registry test asserting the creel module carries TrustedHubsRepositoryReinterpretation, and a hubregistry test for LookupRepository covering a recorded shortname resolving to its origin repository and an unrecorded or ambiguous shortname returning the Lookup error.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/creelconfig/reinterpret.go:272
    summary: "every resolve error drops the entry, including a transient failure such as an unreadable per-user state directory, and the reinterpretation is then recorded as applied so the entry is never retried"
    fix: In RewriteShortnamesToRepositories drop an entry only when resolve fails with hubregistry's none-or-several refusal or a missing-origin reason; have repositoryOfShortname in internal/configreg/configreg.go return the UserStateDir error unchanged and the rewrite return any other resolve error as its error, so hubreconcile does not mark the reinterpretation applied. Add a row in reinterpret_test.go for a resolver that fails with an unrelated error.
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: internal/hubregistry/repository.go:652
    summary: "RepositoryName passes owner and repo through unchecked from the origin URL, and the name is rendered into the agent-facing stamp, so a crafted segment such as one containing parentheses can imitate a stamp"
    fix: In RepositoryName reject an owner or repo that falls outside GitHub's name characters (letters, digits, hyphen, underscore and dot) with an error naming the offending remote, and add a row to the creel hubs test in internal/creelcli/cli_test.go showing such a hub carries no_repository and is untrusted.
---

### [MEDIUM] Pre-post length check ignores the repository now in the stamp

**Location:** internal/creelengine/crosshub.go:36
**Issue:** `validateStampedLine` compares `widestStampedLine` to `boundaryClaimBudget` before a cross-hub post. `widestStampedLine` builds its message with `SendingHub: "hub"` and no `SendingRepository`, while `HubStamp` now prefixes the repository to the stamp. A body that passes the check can exceed the budget on the receiving hub's delivered line.

### [MEDIUM] SendingRepository replay and carry are unpinned

**Location:** internal/creelengine/journal_test.go:802
**Issue:** The field is added to the `inbound` record and folded in `applyInbound`, `applyCarried` and `carryRecords`. Existing replay and carry tests build records without it, and `crosshub_test.go` checks only the live message, so dropping the field from any of the three folds passes every test.

### [MEDIUM] sending_repository envelope field is untested

**Location:** internal/creelcli/cli.go:326
**Issue:** `messageFields` adds `sending_repository` when set. `creelcli/doc.go` documents it as part of the read envelope, but no test asserts it present or absent.

### [MEDIUM] Reinterpretation wiring is untested

**Location:** internal/configreg/configreg.go:164
**Issue:** `reinterpret_test.go` covers the rewrite over a scripted resolver. The registration on the `creel` module, `repositoryOfShortname` and `LookupRepository`, which carry the Goal's "no hand step" behaviour on a real machine, run in no test.

### [LOW] A transient resolve failure drops a trust entry for good

**Location:** internal/creelconfig/reinterpret.go:272
**Issue:** The Goal drops a shortname that "resolves to none". The rewrite also drops it when the per-user state directory cannot be read, and the walk records the reinterpretation as applied, so the entry is lost without a retry. It fails closed, so the harm is a lost trust entry that a line names, not a widened trust.

### [LOW] Repository segments are not checked before they reach the stamp

**Location:** internal/hubregistry/repository.go:652
**Issue:** `githubclient.ParseOwnerRepo` splits the path on `/` and checks nothing else. A repository segment such as `(other hub, trusted orch)` therefore reaches the agent-facing line ahead of the real stamp. The real stamp still follows, and a process of the same user can edit `creel.yaml` anyway (the record's Limit), so this is LOW.

## Rejected

- Lens code-tests F-D (no direct test of `hubDirectory.RepositoryName`): a one-line pass-through that the `Deliver` rows reach through the fake directory and the `creel hubs` row reaches through `hubregistry`; a per-helper test would duplicate that coverage.
- Lens code-affected-set F2 (`log` entry omits the repository): the Goal asks for the name in `lym creel hubs` and the trust stamp only, so adding it to `log` would exceed the answer key.
- Lens code-correctness F-B (repository read twice): the lens itself calls it not a defect.
- `fabricengine/codebinding.go` `HubOriginURL`: the focus asks whether it belongs in scope. It is the one place that finds a hub's prime and reads its origin, `fabricengine` is the owner of git on the warp per PATTERN-fabric-boundary, and the read goes through `gitrepo`, so it is licensed by the Goal's "read from the sending hub's origin remote".
- Declared PATTERN changes: none declared; the `PATTERN-agent-addressing` rule text is updated to the new behaviour and `PATTERN.md`'s one-line entry still holds.
