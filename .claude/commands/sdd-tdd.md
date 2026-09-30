---
model: sonnet
description: Run one TDD cycle (RED → GREEN → REFACTOR → CHALLENGE → STOP)
argument-hint: "<test class or method to drive>"
---

Run ONE TDD cycle for: $ARGUMENTS

Read CLAUDE.md for architecture and testing conventions before writing any code.


## RED — confirm the failure

Run the failing test first. Read the failure message.
Understand WHY it fails before writing any production code.
If the test already passes, STOP — something is wrong.

## GREEN — minimum code to pass
Write the MINIMUM production code to make this one test pass.
Minimum means minimum:
- No extra methods "while we're here"
- No anticipating the next test
- No abstractions until refactoring demands them
- Hard-code if that's all this test requires

Respect architecture boundaries:
- Dto NEVER import org.springframework.* or jakarta.persistence.* packages.
- Controllers NEVER contain business logic.
- Services NEVER contain persistence logic.
- Repositories NEVER contain business logic.
- JPA entities live in model/ and are never exposed over HTTP.
- Domain exceptions live in exception/; map them to HTTP only in
  controller/ (@RestControllerAdvice).

Respect project conventions:
- BigDecimal for ALL money, scale 2, explicit RoundingMode.DOWN.
  Never new BigDecimal(double).
- Records for value objects. No Lombok. Constructor injection only.
- If an entity gains or changes a column, update
  src/test/resources/schema.sql in the SAME cycle — ddl-auto=validate
  means a mismatch fails context startup in every @SpringBootTest.

## REFACTOR — clean up with confidence
All tests are green. Now improve the code:
- Remove duplication
- Extract clear names
- Simplify conditionals
- Check that the code reads like the spec

Run ALL tests after refactoring with `mvn verify` — not just the current one.
(`mvn test` skips the *IT acceptance tests.)
If anything breaks, fix it before moving on.

## CHALLENGE — drive out edge cases
Before stopping, ask yourself:
"What else should this do?"
"What input could break this?"

Consider: zero/empty input, not-found, boundary values, rounding, invalid state, null, negative amounts,
duplicate requests.

Propose at least one edge case to the user.
If approved, that edge case becomes the next RED.

## STOP
Report what you changed:
- Which test is now passing
- What production code you wrote or modified
- What you refactored
- What edge case you propose next

Do NOT write additional tests beyond the one specified.
Do NOT add unrequested features or "improvements".
Do NOT modify any existing test to make it pass — fix the production code instead.

Wait for the user before starting the next cycle.