# lym gate redundant lists a test redundant when its covers never feed a clause its input

`redundant-clause-inputs-unfed` · Note · bug

Found by the redundant-candidates-check trial on internal/boardengine. Equal block coverage plus equal kills of negated conditions does not mean equal inputs.

TestRunLock_BeamKindAndBeamChange is a candidate covered by TestRunLock. Only the candidate changes a beam project's kind or beam alone, so the before.Kind != now.Kind and before.Beam != now.Beam clauses of checkRunLocks are exercised by it alone. TestRunLock's beam rows trip the project, recipe, removal and merge clauses. Negating a clause breaks every accepted beam row in the cover, so the candidate has no unique kill, yet deleting a clause fails nothing else.

TestRunLock_SketchExemptionIsTheWholeNetChange is a candidate covered by TestFabricReplay, TestMakeSketch and TestCheckRunLocks_FamilyProject. Only the candidate feeds a Sketch with a changed title, a changed body or a kept run status.

TestReadRaddleStore is a candidate covered by TestSlugRule_RaddleStore. Only the candidate asserts the decoded Hash and Project, the tolerated unknown key and the exact git checkout command.

Fix to weigh: a clause-deletion mutant for each operand of a boolean chain.