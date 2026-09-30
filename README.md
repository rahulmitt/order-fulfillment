# order-fulfillment

## User Story: Order Fulfillment server

### Story Description
As a registered customer,
I want to place an order for multiple products in my cart,
so that the items are successfully reserved and prepared for shipping

## Development Process

Every user story follows the spec-driven development flow. The rules and conventions behind each
step live in [`CLAUDE.md`](CLAUDE.md) (Development Process, Testing, Architecture), in the step
commands under [`.claude/commands/`](.claude/commands/), and in the per-layer rules under
[`.claude/rules/`](.claude/rules/) (loaded automatically when editing files in that layer).

```mermaid
flowchart TD
    story(["USER STORY<br/>As a #lt;role#gt;, I want ..., so that ..."])

    subgraph S1["STEP 1: DISCOVERY"]
        map["/sdd-discovery<br/>Example Mapping:<br/>rules · examples · counter-examples"]
        questions["Ask open questions<br/>one at a time"]
        present["Present complete spec<br/>⏸ STOP"]
        map --> questions --> present
    end

    approve{"Spec<br/>approved?"}
    save["Save docs/specs/#lt;feature#gt;.md<br/>⏸ STOP"]

    story --> map
    present --> approve
    approve -- "no — revise" --> present
    approve -- "no — new open question" --> questions
    approve -- yes --> save

    subgraph S2["STEP 2: ACCEPTANCE TEST"]
        it["/sdd-acceptance-criteria<br/>Re-read the spec, write #lt;Feature#gt;AcceptanceIT<br/>for the NEXT rule only"]
        itrun["Run it<br/>⏸ STOP"]
        itFails{"Fails for the<br/>right reason?"}
        itFix["Investigate<br/>and fix the test"]
        it --> itrun --> itFails
        itFails -- "no — passes or<br/>wrong failure" --> itFix --> itrun
    end

    subgraph S3["STEP 3: TDD CYCLE"]
        red["/sdd-tdd<br/><b>RED</b> one failing unit test"]
        redFails{"Fails for the<br/>right reason?"}
        redFix["⏸ STOP<br/>investigate"]
        green["<b>GREEN</b><br/>minimum code"]
        refactor["<b>REFACTOR</b><br/>mvn verify (all tests)"]
        outer["<b>OUTER CHECK</b><br/>re-run AcceptanceIT"]
        challenge["<b>CHALLENGE</b><br/>propose edge case<br/>⏸ STOP"]
        red --> redFails
        redFails -- "no — passes or<br/>wrong failure" --> redFix --> red
        redFails -- yes --> green --> refactor --> outer --> challenge
    end

    edgeCase{"Edge case<br/>approved?"}
    ruleGreen{"Acceptance<br/>green?"}
    more{"More rules?"}

    subgraph S4["STEP 4: REVIEW"]
        review["/sdd-review<br/>Read-only report<br/>⏸ STOP"]
        findings{"Findings<br/>to fix?"}
        fix["Fix findings"]
        claudemd["Apply CLAUDE.md updates<br/>only if you agree"]
        review --> findings
        findings -- "yes — other findings" --> fix --> review
        findings -- no --> claudemd
    end

    done(["FEATURE DONE"])

    save --> it
    itFails -- yes --> red
    challenge --> edgeCase
    edgeCase -- "yes — next cycle:<br/>edge case as RED" --> red
    edgeCase -- no --> ruleGreen
    ruleGreen -- "no — next cycle:<br/>next missing behaviour" --> red
    ruleGreen -- yes --> more
    more -- "yes — next rule" --> it
    more -- no --> review
    findings -- "yes — missing behaviour:<br/>new cycle" --> red
    claudemd --> done

    classDef outerLoop fill:#e2d9f3,stroke:#6f42c1,color:#000
    classDef red fill:#f8d7da,stroke:#b02a37,color:#000
    classDef green fill:#d1e7dd,stroke:#146c43,color:#000
    classDef refactor fill:#cfe2ff,stroke:#0a58ca,color:#000
    classDef userChoice fill:#fff3cd,stroke:#b08900,color:#000
    class it,itrun,itFails,itFix,outer,ruleGreen,more outerLoop
    class red,redFails,redFix red
    class green green
    class refactor refactor
    class approve,edgeCase,findings userChoice
```

**Reading the diagram**

| Colour | Meaning |
|---|---|
| 🟪 Purple | **Outer loop** (acceptance level): write a rule's acceptance test, re-run it after every cycle, and move to the next rule once it is green. |
| 🟥 Red / 🟩 Green / 🟦 Blue | **Inner loop** (unit level): RED → GREEN → REFACTOR, one unit test per cycle. |
| 🟨 Yellow | **Your decision**: approving the spec, an edge case, or what to do with review findings. |
| ⬜ Plain | Supporting steps: discovery, CHALLENGE, review. |

- **OUTER CHECK drives the outer loop.** Its result (*Acceptance green?*) decides whether the
  rule needs another inner cycle or is done.
- **⏸ STOP** hands control back to you. Every `/sdd-*` step ends with one, and a test that passes
  when it should fail is always a STOP to investigate.
- **An approved edge case goes first.** It becomes the next RED even when the acceptance test is
  still red; the following OUTER CHECK picks up the remaining missing behaviour. Edge cases can
  also add cycles after the rule's acceptance test is green.
- **Review can send you back.** A missing behaviour becomes a new TDD cycle; other findings are
  fixed and re-reviewed before the feature is done.
