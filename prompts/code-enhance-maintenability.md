# Task: behavior-preserving refactor for maintainability

## Target
<PASTE FILE PATHS HERE>

## Absolute constraint
Not breaking the existing code outranks every refactoring goal. If an improvement cannot be
shown to be safe, do not make it — record it in the out-of-scope list instead.

Observable behavior must be identical before and after: return values, side effects, thrown
errors (type and message), public signatures, ordering, output others depend on. A refactor
that "looks cleaner" but shifts one edge case is a failure, not a trade-off.

## Before changing anything
1. Read the target files in full, plus their call sites.
2. Identify how correctness is verified here: test suite, type checker, linter, build.
   Run them and record the baseline, including pre-existing failures. Do not fix pre-existing
   failures; they are out of scope.
3. Where the code you intend to touch has no coverage, either add characterization tests first,
   or restrict yourself to mechanically safe transformations.

## Permitted transformations, in order of preference
- Rename for clarity, innermost scope first.
- Extract function or method with no logic change.
- Remove code proven unreachable.
- Deduplicate logic that is provably identical.
- Flatten nesting with early returns / guard clauses.
- Replace magic values with named constants.
- Split an oversized module along seams that already exist.
- Add or tighten type annotations where this has no runtime effect.

## Forbidden without explicit approval
- Changing a public API signature or its contract.
- Adding a dependency.
- Restructuring the directory layout.
- Fixing bugs found along the way — record them, do not fix them.
- Performance optimization.
- Reformatting entire files; diff noise hides the real change.
- Altering error-handling semantics, including widening or narrowing a catch.

## Scope extension
You may edit files outside the target list when the edit is a direct consequence of the refactor:
import updates, call sites of a renamed symbol, tests referencing it. Every such file must be
listed in the final report with its causal link. Anything that is not a direct consequence is
out of scope, no matter how tempting.

## Method
Work in small increments. Re-run the verification from step 2 after each one. If it turns red,
revert that increment rather than patching over it. Never accumulate unverified changes.

## Final report — these sections, in this order

### Modifications
Per file: path, what changed, how it improves maintainability.

### Files touched outside the requested scope
Per file: path, what changed, the direct causal link to the refactor.

### Verification
Commands run, baseline result vs final result.

### Noticed but out of scope
Bugs, design smells, code that looks dead but could not be proven dead, missing coverage,
risky patterns. For each: location, description, why it was left alone. No fixes.

If nothing can be improved safely, state that and deliver only the out-of-scope list.