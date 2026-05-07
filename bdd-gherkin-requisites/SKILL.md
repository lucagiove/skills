---
name: bdd-gherkin-requisites
description: BDD scenario design for capturing acceptance requirements - Given-When-Then structure, scenario writing rules, categorization, scenario outlines, anti-patterns, and living documentation. Use when authoring acceptance criteria, drafting feature files, defining requirements in BDD form, or specifying behaviour before implementation. Implementation-agnostic (no test framework or coding guidance).
---

# BDD Gherkin Requisites

Scenario-design-only companion to a BDD workflow. Use this skill to capture
behaviour as Gherkin requisites; pair it with a separate implementation skill
(e.g. an outside-in acceptance-test skill) when you need to wire scenarios up
to code.

## Core Philosophy

Test units of behaviour, not units of code. Acceptance scenarios validate
business outcomes through public interfaces, decoupled from implementation.

## Outside-In Double-Loop TDD

Scenarios authored with this skill form the **outer loop** of Outside-In TDD.
Development starts from user perspective and drives inward.

**Outer loop (acceptance / BDD)** — Hours to days. User perspective, business
language. Defines "done". Scenarios describe user goals and observable
outcomes, not internals. A failing outer-loop scenario is the starting signal
for implementation.

**Inner loop (unit / TDD)** — Minutes. Developer perspective, technical terms.
Owned by whoever writes the implementation; out of scope for this skill.

Workflow:

1. Write a failing acceptance scenario from the user perspective (outer loop — outside).
2. Implementer drops to the inner loop: unit tests to build components (inside).
3. Iterate the inner loop until the acceptance scenario passes.
4. The passing acceptance scenario proves user value delivered.
5. Repeat for the next behaviour.

Outer loop defines WHAT users need (outside). Inner loop drives HOW to build it (inside).

## Given-When-Then Structure

```gherkin
Scenario: [Business-focused title describing one behaviour]
  Given [preconditions - system state in business terms]
  When [single user action or business event]
  Then [observable business outcome]
```

### Scenario Writing Rules

**Rule 1: One scenario, one behaviour** — Split multi-behaviour scenarios.

**Rule 2: Declarative, not imperative** — Business outcomes, not UI
interactions. "When I log in with valid credentials" not "When I click Login
button and enter email."

**Rule 3: Concrete examples, not abstractions** — "Given my account balance is
$100.00" not "Given the user has sufficient funds."

**Rule 4: Keep scenarios short (3-5 steps)** — Longer means testing multiple
behaviours or irrelevant details.

**Rule 5: Background for shared Given steps only** — Only Given steps.
Actions/validations belong in scenarios.

## Scenario Categorization

- **Happy path**: Primary successful workflows
- **Error path**: Invalid inputs, failures, unauthorized access (target 40%+ of scenarios)
- **Edge case**: Boundary conditions, unusual but valid behaviour
- **Integration**: Cross-component / system interactions

### Golden Path + Key Alternatives

Per capability:

1. Happy path (most common success)
2. Alternative paths (valid but less common)
3. Error paths (invalid inputs, constraint violations)

Select representative examples that reveal different business rules. Do not
test every combination.

### Scenario Outlines for Boundary Testing

```gherkin
Scenario Outline: Account minimum balance validation
  Given I have an account with balance $<initial_balance>
  When I attempt to withdraw $<withdrawal_amount>
  Then the withdrawal is <result>

  Examples: Valid withdrawals
    | initial_balance | withdrawal_amount | result   |
    | 100.00          | 50.00             | accepted |
    | 25.00           | 25.00             | accepted |

  Examples: Invalid withdrawals
    | initial_balance | withdrawal_amount | result                        |
    | 100.00          | 101.00            | rejected (insufficient funds) |
```

Use outlines for boundary conditions and calculation variations. Avoid them
when scenarios diverge structurally.

## Anti-Patterns

| Anti-Pattern                                    | Fix                                          |
| ----------------------------------------------- | -------------------------------------------- |
| Testing through UI                              | Specify behaviour at the service / API layer |
| Multiple WHEN actions                           | Split into separate scenarios                |
| Conjunction steps ("Given A and B" as one step) | Break into atomic steps                      |
| Incidental details                              | Include only behaviour-relevant info         |
| Technical jargon in scenarios                   | Business domain language                     |
| Abstract scenarios                              | Concrete values, specific examples           |
| Rambling scenarios (8+ steps)                   | Extract to 3-5 focused steps                 |

## Living Documentation

Scenarios serve a dual purpose: executable specification and living
documentation. Organisation: **Business Goal > Capability > Feature > Scenario
> Test**. Each scenario traces to a business capability. Stakeholders see
which capabilities are implemented, tested, and passing.

### Documentation-Grade Scenarios

Replace HTTP verbs with business actions, JSON payloads with domain concepts,
status codes with business outcomes. Add context about WHO and WHY — not
about wire formats.

## Authoring Checklist

Before a scenario set is "done":

- [ ] Each scenario covers exactly one behaviour
- [ ] Steps are declarative (business language), not imperative (UI clicks)
- [ ] Concrete values used everywhere; no "sufficient", "valid", "some"
- [ ] 3–5 steps per scenario (extract or split if longer)
- [ ] Backgrounds contain only Given steps
- [ ] Per capability: at least one happy path, one alternative, one error path
- [ ] Boundary conditions captured via Scenario Outline where applicable
- [ ] No HTTP verbs, JSON, status codes, or framework jargon in step text
- [ ] Each scenario traces back to a named business capability
