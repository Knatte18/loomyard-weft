```yaml
lens: interactions
outcome: reported
```

Task: interactions lens over the decision record's pairs of decisions.

**Pairs checked**
- D1 empty list × D2 fresh launch × D3 step 2: broken. With the template default `[]`, the fresh launch still runs D3's pick, and D3's no-candidate rule fails it.
- D1 empty list × D2 resume with no record × D4 default unset: broken. With the feature off, the resume still writes a record, which then overrides the `env` module's value.
- D2 resume search set × D6 known directories: broken. D2 cites D6 but searches a narrower set.
- D2 fresh pick × D2 resume with no record × D7 pre-change sessions: broken. The outcome depends on which launch comes first.
- D2 resume record write × D3 step 1 lock and re-check: broken. The resume's write has no stated lock, so it can overwrite a pick.
- D2 empty account name × D5 listed-account condition: held. An empty name gets no status line.
- D2 name-keyed record × D1 list edits × D5 status line: broken (low). A rename or reuse of a name goes wrong.
- D2 record kept after its account leaves the list × D3 counter: held. Counts for unlisted names are ignored, and seeding uses only listed accounts.
- D3 eligibility × D5 headroom definition: held.
- D3 step 4 fallback × D5 windows: broken (low). A candidate with several blocking windows is ambiguous, and fallback ties are unstated.
- D2 "reed relaunch reads the record" × D4 "relaunch replays the stored command": broken (low).
- D4 export order × `env` module × D6 reader fallback: held when no record exists.
- D5 per-account usage × D3 per-hub counter: held. Both are stated per hub.
- D6 plugin lookup × D9 plugins link: held.
- D7 × D2 for prime and beam worktrees: broken. This is folded into the ordering finding.
- D8 × D2 and D4 resume keeping its directory: held.
- Constraints × decisions: held. Shuttle seam × D4 and D5, lym-files × D2, D3 and D5, config-layers × D1, tolerant-records × D2, D3 and D5, cli-cobra × D5.
- Criteria: the D1 "empty and no record" criterion and the D2 "resume no record records default" criterion are contradicted by the empty-list and `env` findings. The other criteria are consistent with their decisions.

**Findings**

1. BLOCKING · design · D2 "How a session launch reads it", D3 steps 2 and 5, D4 first bullet.
   With an empty or absent `claude.accounts`, D2 and D3 still act.
   - **Fresh launch:** a fresh launch with no record "picks an account (D3)", and D3 step 2 fails the launch when there are no candidates.
   - **Resume with no record:** a resume with no record writes a record of the transcript's directory, or of the default directory.
   - **Contradiction:** D4 then unsets `CLAUDE_CONFIG_DIR` for a default record, overriding an `env` module value. Scope says hubs without accounts and standalone runs keep today's behaviour, and D4 and the D1 criterion say "with no record and no accounts, exports nothing".

   No decision gates the pick or the record write on a non-empty list.
   Evidence: decision-record.md:64-74, :89-93, :114-116, :26, :231.

2. BLOCKING · design · D2 "fresh launch with no record" against "resume launch with no record", and D7.
   - **Ordering:** whether a pre-change session can resume depends on launch order. If any fresh session in a worktree, such as the dobby, a reviewer or a fixer, launches before the pre-change orch or writer session resumes, the fresh launch picks account B and writes the record.
   - **Resume fails:** the resume then finds a record whose directory exists, keeps it (D2's first bullet), and launches under B. Its transcript is in the default directory, so the resume check refuses, as D2's own rationale says.
   - **Concurrency:** the resume's record write is not stated to run under D3's lock with the step 1 "existing record wins" re-check. A concurrent resume and fresh pick can each write a record, and the later write moves the worktree's sessions to the other directory.

   D7's "Their sessions started before this change resume and record the default directory" holds only when the resume comes first.
   Evidence: decision-record.md:66-71, :78-79, :88, :179.

3. MEDIUM · consistency · D2 resume search set against D6 known directories.
   - **Narrower set:** D2 searches "the listed accounts' directories and the default directory (D6)". D6's known set also holds the `env` module's `CLAUDE_CONFIG_DIR` and the process's own value.
   - **Effect:** a hub that sets `CLAUDE_CONFIG_DIR` through the `env` module today, which the board's "Built since" section says it can, finds none of its pre-change transcripts. It records the default directory, D4 then unsets the variable, and every such resume is refused.

   The same gap falsifies D7's pre-change claim and the D2 criterion that a resume with no record records the default directory.
   Evidence: decision-record.md:71-73, :165, :114-116; board entry "Built since".

4. LOW · design · D2 record (name and directory) × D1 list edits × D5.
   The record stores the account name, and D5 gates and names the status line by that name.
   - **Rename:** if an account is renamed with the same directory, the worktree loses its status line, so its headroom goes unrecorded.
   - **Name reuse:** if a name is reused for another directory, usage from this worktree's directory is written under the other account.

   No decision says whether the name or the directory is authoritative after a list edit.
   Evidence: decision-record.md:61, :130-131, :155.

5. LOW · consistency · D3 step 4 against D5.
   - **Ambiguity:** "the candidate whose blocking windows all reset soonest" does not say how a candidate with several blocking windows is ranked. It should be the latest `resets_at` among its blocking windows.
   - **Ties:** fallback ties are unstated. Step 3 states its tie rule, but step 4 has none.

   The D3 fallback criterion inherits both gaps.
   Evidence: decision-record.md:94-96, :143, :239.

6. LOW · consistency · D2 last paragraph against D4 third bullet.
   - **D2:** a reed relaunch "read[s] the record and keep[s] the directory".
   - **D4:** a reed relaunch replays the stored command, with the variable baked in, and reads no record.

   After the operator removes or replaces the record, which is the only way D7 and the Auto-mode assumptions give to change a worktree's account, a reed relaunch still runs under the old directory. A gone directory is therefore not refused as D2's resume rule requires.
   Evidence: decision-record.md:76, :118, :218.

Tool calls: none.
