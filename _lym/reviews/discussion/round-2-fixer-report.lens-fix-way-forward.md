```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: check that every refusal, error, step, row or test the fix added can be reached and leads somewhere.

Ways forward the fix introduced:
1. D2: the build fails naming the key when the board's HEAD differs from the records bare's branch tip.
2. D2 criterion: a build that commits into the records bare after the board's push fails naming the key.
3. D7: a step that commits straight into a bare stays in the test after the copy; `endedHub`'s `originBoard` runs on the copy.
4. D1: a directory that does not exist is an error with no walk-up, so the `junction.go` and `scratchlink.go` fallbacks fire.
5. D1 criterion: `List` from a subdirectory equals `List` from the root, and `List` on a missing directory errors.
6. D2 Bound: a test that installs a process-global seam takes its copies before installing it, and a build calls no seam setter or once-only registration.
7. D7 review criteria: the two checks for item 6.
8. D3 criterion: a copy of a shape, or of a registration whose build commits what it writes, has a clean board.

Items 1–5, 7 and 8 can be reached and lead somewhere.
- Item 3 works because a copy's fold and push happen inside `CopyHub`, so an `originBoard` commit made afterwards survives. The test can get the branch from the copy's `BoardDir`.
- Item 4: `junction.go:624-630` and `scratchlink.go:175-180` fall back to `List(l.WorktreePath())` on any error from `List`, then mark everything as wired (junction) or answer true (scratchlink). A missing-directory error reaches those fallbacks.

Finding:

- LOW. Changed text, D2 Bound: "A build also runs under whatever process-global seams and registrations are installed when its first caller reaches it … So a build calls no package-global seam setter or once-only registration, and a test that installs a process-global seam takes its copies before installing it."
  Summary: the Bound names a hazard from registrations that are already installed, but the rule it gives handles only seams that get restored. A build that runs after another test's once-only registration has no way forward.
  Evidence: `boardkit.InstallReplay` is never undone (`internal/testkit/boardkit/boardkit.go:26`, "The registration is never restored"). `TestReconcileOrigin_Ended` calls it at its top (`originreconcileended_integration_test.go:107`), before any `endedHub` copy. No ordering inside one test can put copies before a registration that an earlier test in the binary already made. So "takes its copies before installing it" cannot be done for registrations, and whether a template was built with or without the replayer depends on test order. A way forward would be either "a build reaches nothing a once-only registration changes" or "a build depends on no once-only registration's presence". The D7 review criteria would need to check whichever is chosen.

Tool calls:
- `sed` of `internal/fabricengine/junction.go:618-632` and `internal/fabricengine/scratchlink.go:170-182`: my context had only grep hits for the `List` calls. I needed the code around them to confirm the fallback fires on any error, including the new missing-directory error.
