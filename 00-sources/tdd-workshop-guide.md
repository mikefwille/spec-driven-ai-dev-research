---
type: source
date: 2026-01-08
origin: "[[2026-01-08 TDD workshop w_ Jonathan Turner - o5lpkev2bdrr49ov49c8tjmpig]]"  # _areas/work/marq-command-center/transcripts
---

# TDD Workshop - Jonathan Turner

Workshop conducted at Marq on January 8, 2026, led by Jonathan Turner (Chief Architect, 20+ years software engineering, Utah Software Craftsmanship Group organizer for 11 years).

## What Test-Driven Development Is (And Isn't)

TDD is **not** just writing tests. Writing tests is a part of TDD, but TDD is a fundamentally different discipline from "write code, then write tests for it." TDD is an iterative design process where tests drive the shape of production code.

A common misconception is that any code with tests was developed using TDD. The distinction lies in the process: TDD means writing the test *before* the production code and proceeding in small, deliberate increments.

## The Three Rules of TDD

These rules were defined by Robert Martin (Uncle Bob):

1. **You are not allowed to write any production code unless it is to make a failing unit test pass.** Before you write production code, you must write a test for it first.

2. **You are not allowed to write any more of a unit test than is sufficient to fail.** Compilation failures count as failures. You don't write a batch of tests and then a batch of production code. You write one test, then enough code to pass it.

3. **You are not allowed to write any more production code than is sufficient to pass the one failing unit test.** You can't write one test and then go implement an entire feature. Write just enough production code to make the single failing test pass.

The key takeaway: TDD is an *iterative* process. Write a test, write a little production code. Write another test, write a little more production code. Back and forth, always in small increments.

## The Red-Green-Refactor Cycle

This is the core rhythm of TDD practice:

### Red (Write a Failing Test)
- Write a new test before any production code exists for that behavior
- Run it and confirm it fails
- Verify it fails *for the expected reason* -- this ensures the test is actually testing what you intend

### Green (Make It Pass)
- Write the **simplest production code** that makes the failing test pass
- All existing tests must also still pass
- Don't over-engineer at this stage; just make it work

### Refactor (Clean Up)
- This phase is **optional** -- only refactor if there's something to improve
- Change the form of the code without changing its function
- Rename things, remove duplication, restructure methods
- Do NOT add new features or functionality during refactoring
- Run tests after refactoring to confirm behavior is preserved

Then repeat the cycle: write another failing test, make it pass, refactor.

## Philosophy and Key Principles

### "Do the Simplest Thing That Could Possibly Work" -- Ward Cunningham

This heuristic guides the Green phase. When making a test pass, implement the absolute simplest solution, even if it feels trivially dumb. Return a hard-coded value. Add an if-statement. Copy-paste a line and modify it.

This feels wrong to experienced developers. But the simplicity is the point: it keeps you moving forward in tiny, verifiable steps rather than leaping to a complex solution that may be wrong.

### "Make It Run, Make It Right, Make It Fast" -- Kent Beck

This maps directly to Red-Green-Refactor:
- **Make it run**: Get a failing test to pass (Green phase)
- **Make it right**: Refactor the code to a clean design (Refactor phase)
- **Make it fast**: Optimize later, only when needed

The priority order matters. Correctness comes first. Elegance comes second. Performance comes last.

### Legacy Code -- Michael Feathers

From *Working Effectively with Legacy Code*: "Code without tests is bad code. It doesn't matter how well written it is, it doesn't matter how pretty or object-oriented or well encapsulated it is. With tests, we can change the behavior of code quickly and verifiably. Without them, we really don't know if our code is getting better or worse."

Feathers' definition of legacy code: **any code that doesn't have tests.** Without tests, you're "typing and praying" -- which is not a sound development strategy.

### TDD as a Design Practice -- Sandro Mancuso

From *The Software Craftsman*: "Although Test-Driven Development has 'test' in its name, TDD is actually a design practice."

People fixate on the "test" in TDD. But the deeper purpose is to influence the **design** and **shape** of your code. Writing the test first forces you to think about the interface and usage of your code before implementing it.

### TDD and Productivity -- Dave Farley

From *Modern Software Engineering*, citing DORA/State of DevOps research: "High performing teams that employ techniques like TDD, continuous integration and continuous delivery spend 44% more time on useful work. These teams are more productive than the norm while at the same time producing higher quality outcomes."

You can have quality AND speed. They are not inherently in tension.

## Why TDD Matters

1. **Correctness**: You end up with tests that prove your code does what you intend. You know your code is correct because you have evidence.

2. **Testable code by definition**: Because you write the test first, the production code is testable by construction. You never hit the situation of "this code is too hard to test, so I'm not going to."

3. **Maintainability**: Without TDD, projects tend to follow a depressing arc: they start with excitement and end as a tangled mess that developers dread touching. TDD reverses that trajectory. Tests accumulate over time, giving you increasing confidence to modify and improve the codebase. Kent Beck: "TDD is for geeks who form emotional attachments to code." His goal: "to feel better about a project after a year than I did in the starry-eyed beginning, and TDD helps me achieve this."

4. **Simpler, better-designed code**: The iterative process and the constraint of simplicity often lead to solutions that are more elegant than what you'd arrive at by just starting to code. The Roman numeral kata demonstrates this well -- TDD can lead you to a 3-line recursive solution with a mapping table, which is far simpler than the nested-loop solution most developers initially envision.

5. **Problem decomposition**: TDD helps break big, overwhelming problems into tiny, manageable steps. You don't have to solve the whole problem at once. You just need to add one more test and make it pass.

6. **Faster, higher-quality delivery**: Research backs this up. Teams using TDD along with CI/CD produce better outcomes faster.

## The Three TDD Activities

When working through TDD, you'll naturally use three different modes:

### 1. Obvious Implementation
If you know exactly what code to write, just write a test for it and implement it. No need to take artificially tiny steps when the solution is clear. Write the test, write the code, move on.

### 2. Fake It
Make the test pass with an implementation that isn't really the solution you want, but it satisfies the test. Return a hard-coded value. Add a simple if-statement. This is useful when you're not sure what the final implementation looks like yet.

**Rule of thumb**: If you don't know what to do next, Fake It.

### 3. Triangulate
You have a partial solution that isn't general enough yet, and you're using additional test cases to drive it toward the real solution. Each new test forces the implementation to become slightly more general.

The natural progression is often: Fake It -> Triangulate -> Obvious Implementation, as understanding of the problem grows.

## Practical Patterns

### Guard Clauses
A common TDD pattern: you implement some functionality, then when a new test case requires different behavior, you add a guard clause to limit when the original code executes. Example:
- First: `return ""`
- After adding the "1 returns I" test: `if (arabic === 0) return ""; return "I";`

The original behavior gets scoped to its specific case, and the new behavior becomes the default (or gets its own guard).

### Table-Driven Tests
As you accumulate test cases that follow the same pattern (same structure, different inputs/outputs), refactor them into a single parameterized test with a data table. This reduces duplication in test code and makes adding new test cases trivial.

In Vitest: use `it.each()` or `test.each()` with an array of test cases.
In Jest: same pattern with `it.each()`.
In C# (xUnit/NUnit): use `[TestCase]` or `[InlineData]` attributes.

### Make the Implicit Explicit
When you notice a relationship that only exists in your head -- for example, that `1` maps to `I` and `5` maps to `V` -- make it explicit in the code as a data structure. This is often the moment where procedural logic (a chain of if-statements) transforms into a mapping table plus a simple algorithm. The Roman numeral kata demonstrates this: scattered guard clauses became a `mappings` list once the pattern was named and extracted.

### Choosing Your Next Test
When deciding what test to write next, consider:
- **Fake It path**: Pick a case that's easy, just adds another simple mapping (e.g., testing 10 when you already handle 1 and 5)
- **Triangulate path**: Pick a case that forces your implementation to become more general (e.g., testing 2 when you only handle single-character returns)
- **Avoid the scary case**: If a test case (like subtractive Roman numerals) feels overwhelming, skip it for now and handle simpler cases first. Build up to the hard cases.

## The Roman Numeral Kata -- Walkthrough

This kata converts Arabic integers (1-3999) to Roman numeral strings. The key progression:

### Phase 1: Fake It (Tests for 0, 1, 5, 10)
Each case was handled with simple guard clauses and hard-coded returns:
```
if (arabic === 0) return ""
if (arabic === 5) return "V"
return "X"  // default
```
This is unimpressive but correct for the test cases that exist.

### Phase 2: Triangulate (Test for 2)
Adding `2 -> "II"` forces the code to handle multi-character output. This led to discovering recursion:
```
if (arabic >= 1 && arabic < 5) return "I" + convertToRoman(arabic - 1)
```
The duplication between the "1" line and the "2" line signaled the need for generalization.

### Phase 3: Refactor -- The Mapping Emerges
The duplication between lines handling 1 and 5 led to extracting an explicit mapping:
```
mappings = [[10, "X"], [5, "V"], [1, "I"]]
```
With a generic algorithm: find the largest mapping that fits, take its Roman character, and recurse on the remainder:
```
mapping = mappings.find(m => m.arabic <= arabic)
return mapping.roman + convertToRoman(arabic - mapping.arabic)
```
Three lines of production code plus a mapping table. Vastly simpler than nested loops.

### Phase 4: Extend (100, 500, 1000)
Adding higher values was trivial -- just add entries to the mapping table. No production code changes needed.

### Phase 5: Subtractive Cases (4, 9, 40, 90, 400, 900)
The "scary" cases turned out to be simple: just add them as additional entries in the mapping table (e.g., `[4, "IV"]`, `[9, "IX"]`, `[40, "XL"]`). The algorithm handled them automatically.

### The Result
Final implementation: ~3 lines of logic + a 13-entry mapping table that handles all Roman numerals from 1 to 3999. Most developers who "just start coding" arrive at something significantly more complex.

The mapping table in descending order:
```
1000 -> M, 900 -> CM, 500 -> D, 400 -> CD,
100 -> C, 90 -> XC, 50 -> L, 40 -> XL,
10 -> X, 9 -> IX, 5 -> V, 4 -> IV, 1 -> I
```

## Key Takeaways

1. **Trust the process.** Early TDD steps feel trivial and pointless. You will question whether it's worth it. The payoff comes as the solution emerges from the accumulation of small steps.

2. **Small steps are a tool, not a rule.** Take small steps when you're unsure. Take bigger steps when the implementation is obvious. If a big step goes wrong, fall back to smaller steps.

3. **Duplication is a signal.** When you copy-paste and modify a line, that's a signal to refactor. TDD naturally surfaces duplication and gives you the safety net (tests) to eliminate it.

4. **Tests are code too.** Maintain your test code. Refactor tests to remove duplication (e.g., table-driven tests). Keep them readable.

5. **You can always verify.** Not sure if a case works? Add a test and ask the computer. If it passes, you can keep or delete the test. Uncertainty is cheap to resolve.

6. **TDD often leads to simpler solutions.** The constraint of incremental development and forced simplicity can produce designs you wouldn't have arrived at by "just coding."

7. **TDD doesn't guarantee the optimal solution, but it gives you the safety net to find it.** You might end up with a working-but-inelegant solution -- a loop-based approach instead of a clean recursive one, for example. That's fine. The tests you've built give you the confidence to refactor toward a better design whenever the insight strikes -- whether that's five minutes later or in the shower three days from now. A "worse" solution with tests beats a "better" one without them, because you can always improve the former and never safely touch the latter.

## Recommended Reading

- **Test-Driven Development by Example** -- Kent Beck (the foundational TDD book)
- **Working Effectively with Legacy Code** -- Michael Feathers (getting untested code under test)
- **The Software Craftsman** -- Sandro Mancuso (professional software development philosophy)
- **Modern Software Engineering** -- Dave Farley (engineering practices including TDD, CI/CD)
- **Accelerate** -- Nicole Forsgren, Jez Humble, Gene Kim (research on high-performing engineering teams and DORA metrics)

---

*Based on the TDD Workshop led by Jonathan Turner at Marq, January 8, 2026. Attendees: mwille@marq.com, btraylor@marq.com, productdev@marq.com, sanderson@marq.com, jonathan.m.turner@gmail.com.*
