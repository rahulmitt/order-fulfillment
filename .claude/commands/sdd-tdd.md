---
model: sonnet
description: Run one TDD cycle (RED → GREEN → REFACTOR → CHALLENGE → STOP)
argument-hint: "<acceptance test / behaviour to drive, e.g. PlaceOrderAcceptanceIT Rule 1 example 2>"
---

Run ONE TDD cycle for: $ARGUMENTS

Read CLAUDE.md for architecture and testing conventions before writing any code.


## RED — write ONE failing unit test

Run the red acceptance test (`mvn -Dit.test=<Feature>AcceptanceIT verify`) and read
the failure. Pick the next SMALLEST behaviour it needs that doesn't exist yet.

Write ONE unit test (`*Test`) for that behaviour, in the same package as the class
under test (under `src/test/java`). Choose the right level:
- Service — plain JUnit 5 + Mockito, repositories mocked, no Spring context.
  Business rules belong here (e.g. customer not found, customer not active).
- Controller — `@WebMvcTest` with the service mocked. HTTP mapping only:
  status codes, `@Valid`, `@RestControllerAdvice` error bodies.
- Repository — `@DataJpaTest`, only when there is a custom query.

Run it with `mvn -Dtest=<Class>#<method> test`. Read the failure message.
Understand WHY it fails before writing any production code.
If the test already passes, STOP — something is wrong.

## GREEN — minimum code to pass
Write the MINIMUM production code to make this one unit test pass.
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

## OUTER CHECK — is the acceptance example green?
Re-run the acceptance test: `mvn -Dit.test=<Feature>AcceptanceIT verify`.
- Green → the targeted example is done.
- Still red → name the behaviour that is still missing. It becomes the
  next cycle's RED (a new unit test). Do NOT start it now.

## CHALLENGE — drive out edge cases
Before stopping, ask yourself:
"What else should this do?"
"What input could break this?"

Consider: zero/empty input, not-found, boundary values, rounding, invalid state, null, negative amounts,
duplicate requests.

Propose at least one edge case to the user.
If approved, that edge case becomes the next RED — as a unit test.

## STOP
Report what you changed:
- Which unit test you added, and that it is now passing
- Acceptance test state: which examples are green / still red
- What production code you wrote or modified
- What you refactored
- What edge case you propose next

Do NOT write more than ONE new unit test per cycle.
Do NOT modify the acceptance test.
Do NOT add unrequested features or "improvements".
Do NOT modify any existing test to make it pass — fix the production code instead.

Wait for the user before starting the next cycle.