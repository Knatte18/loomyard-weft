# Plan-Review round 1 fix: proposed Delete glyphs the plan gate refuses

F2's proposed fix put method Delete glyphs (`probingBurlerRunner.Run`, `.ProbeRound`, `.Resume`) on card 4.
`delete-before-reference` matches a method name as text, so `.Run` hit every later card's Edit code and blocked the plan.
Dropping the method glyphs made `caller-uncovered` block on the method bodies instead.
The only admissible shape was moving the deletion to the last card.
The rubric says a reviewer proposes only fixes the checks admit, but finding that out took two validator runs.
A reviewer that runs `lyx loom validate-plan` against its proposed target changes would catch this before the fixer does.

F3's proposed glyphs named the function after or near each cited comment, not the one that encloses it, so each had to be located again.
