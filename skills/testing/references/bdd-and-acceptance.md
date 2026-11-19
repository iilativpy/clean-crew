# Behavior-driven development and acceptance scenarios

Open when turning a requirement into examples, writing Given/When/Then scenarios, automating
acceptance tests, or deciding what level a test belongs at.

## What BDD is, in short
A set of practices that help a team build the right thing by describing behavior with **concrete
examples in a shared language**, then keeping those examples as **executable specifications** and
**living documentation**. It began as a way to teach and practice TDD ("where do I start, what do I
test next?") and brings the business into the loop. Its focus is on **behavior in business terms**, not
on the methods of a class.
- **TDD** works at the unit level: it specifies, designs and verifies code with tests written first.
- **BDD** adds a **conversation and an outer loop**: examples of business behavior drive acceptance
  scenarios, which in turn drive unit-level specifications. They are the same discipline at different
  scales.

## From a vague story to examples
1. **Start from the goal.** Why does anyone want this (the business value), who benefits, and what
   capability delivers it? A story states *In order to ... As a ... I want ...*.
2. **Ask for concrete examples** of the behavior, with real values: "A standard member flies Sydney to
   Melbourne (878 km): how many points?" Examples expose assumptions, ambiguity and missing rules faster
   than abstract statements.
3. **Look for edge cases and counter-examples**: What if the value is exactly at the limit? Empty? The
   first time? The wrong user? Each is another example. Record **open questions** instead of guessing.
4. **Keep the examples** as the acceptance criteria. Collect several per rule, covering the happy case, the
   edges and the rejections.
5. **Do not commit to details too early.** Discover as you build; leave questions open until the last
   responsible moment, and capture what you learn as new examples.

## Writing a scenario
```gherkin
Feature: Earning Frequent Flyer points from flights
  In order to encourage travellers to fly with us
  As the sales manager
  I want travellers to earn points when they fly

  Scenario: Earning standard points from an economy flight
    Given the flying distance between Sydney and Melbourne is 878 km
    And I am a standard Frequent Flyer member
    When I fly from Sydney to Melbourne
    Then I should earn 439 points
```
- **Feature title**: an activity a user or stakeholder wants to perform ("Earning points from flights"),
  not a module ("Point management"). A short description states the business value.
- **Given** the context: the state the system must be in. **When** the one action or event. **Then** the
  observable outcome. **And/But** extend the previous part.
- **Scenario titles** state the example in business words.
- **Scenario Outline + Examples table** for the same rule with many data rows:
  ```gherkin
  Scenario Outline: Status upgrades
    Given a <initialStatus> member with <points> status points
    When they earn <extra> more points
    Then their status is <finalStatus>
    Examples:
      | initialStatus | points | extra | finalStatus |
      | Bronze        | 0      | 300   | Silver      |
      | Silver        | 0      | 700   | Gold        |
  ```
- **Feature files** are plain text, grouped by feature, in version control next to the code; **tags** mark
  the type (smoke, slow, wip) and the story or release.

## Expressive scenarios: patterns
- **Given** puts the application in the right state quickly, in business terms. It may use shortcuts (set
  up data through a back-end service) but must read as the *business context* ("Given I am logged in as an
  administrator"), not the mechanism ("Given an admin row exists in the database").
- Put **preconditions in the past tense** (they already happened) and the **action in the present**.
- **Given contains all and only the preconditions** that matter to the behavior under test.
- **When** is **one action**, stated as what the user does (what), not the clicks (how).
- **Then** states **observable outcomes**, a few related facts about one result.
- **One behavior per scenario**; a scenario whose purpose needs two sentences tests two behaviors.
- **Reuse data in the Then** by referring to what the Given set up instead of repeating tables.
- **Use plain domain language**, not field names, ids, selectors or SQL.

## Anti-patterns in scenarios
| Anti-pattern | Why it hurts | Fix |
|---|---|---|
| **Imperative UI scripts**: "enters 'Bill' in the first name field, clicks submit..." | Brittle (breaks on layout change), unreadable, hides the rule | One high-level step ("When Bill registers with valid details"); put clicks in the technical layer |
| **Technical details in Given** (database rows, ids) | Couples the spec to implementation | State the business context |
| **Unclear what is being tested** (log-in, navigation and the real behavior mixed) | The reader cannot find the point; failure meaning is vague | Hide incidental steps; one behavior |
| **Scenario too long** (dozens of steps) | Nobody reads it; hard to maintain | Merge steps; split scenarios; use tables |
| **Several behaviors per scenario** | Failure does not say which | One behavior per scenario |
| **Dependent scenarios** (one expects another's data) | Order dependence, flaky | Each scenario sets up its own context |
| **Incidental data shown** | Noise hides what matters | Show only the values that matter; default the rest |
| **Scenarios that duplicate unit tests** | Slow, redundant | Test each rule at the lowest level that proves it |
| **Vague outcomes** ("Then it works") | Not verifiable | State the exact observable result |

## Step definitions and automation
- A **step definition** maps a sentence of a scenario to code. Keep each step small, delegate the work
  to a **business-level helper or page object**, and keep **no logic** in the step.
- **Reuse steps** across scenarios; avoid many near-duplicate steps phrased slightly differently.
- **Share state between steps carefully** (a scenario context object), not through globals; reset it for
  every scenario.
- **Layered abstraction** keeps acceptance tests stable:
  1. **Business rules layer**: readable scenarios stating the requirement.
  2. **Business flow layer**: reusable steps expressing user tasks ("registers", "adds to basket").
  3. **Technical layer**: how it is done (page objects, API client, database helper).
  A change in a screen then touches one place, not hundreds of scenarios.
- **Set-up should be automatic and fast:** create test data through APIs or builders, not by clicking;
  isolate each scenario; clean up.

## Which level proves what
- **UI tests** check the user's journey, rendering, form validation messages and the wiring of the screens.
  They are slow and fragile: keep them few.
- **Non-UI acceptance tests** (service or API level) check business rules directly against the layer that
  owns them: faster and more stable. Prefer them for rules, calculations and workflows.
- **Unit-level specifications** (TDD) drive the classes that implement each step; high-level criteria fan
  out into many low-level ones as you build.
- If business logic is scattered through the UI layers, testing it needs the UI: a design smell worth
  fixing, as clean separation makes rules testable below the UI.
- **Non-functional requirements** (performance, security, accessibility) need measurable criteria too:
  write them as scenarios or checks with a number ("responds within 2 seconds for 100 concurrent users").

## The BDD workflow with TDD underneath
1. Agree **examples** for the story. Write the **scenario** (fails: not implemented).
2. **Automate the steps** to see the scenario fail for the right reason.
3. Implement one step's worth of behavior **test-first at unit level**: a failing unit-level statement,
   simplest code, refactor. Discover collaborators and design as you go.
4. Repeat until the scenario passes. Refactor.
5. Add the next example (edge cases, rejections).
Both layers stay green; the acceptance scenario says **done**.

## Living documentation
- Executable specifications are documentation that cannot go stale: if the behavior changes, the build
  fails. Publish the results (a report grouped by feature and status) so non-developers can read what
  the system does and which features are done, in progress or pending.
- Organize scenarios by **feature and capability**, not by test class. Keep a short description of the
  goal on each feature.
- **Run them in the build** (continuous integration). Fast acceptance tests (non-UI, parallel) run on
  every commit; slower end-to-end ones run on a schedule or before release.
- Technical living documentation (the unit-level specifications, architecture rules as tests) serves
  developers the same way.

## When BDD is a poor fit
- When there is no one to have the conversation with, scenarios become test scripts written by
  developers; the benefit is smaller, though the structure still helps.
- For purely technical components with no business-visible behavior, plain unit tests are enough.
- A heavy scenario suite with many UI steps becomes a maintenance burden: keep it lean and layered.
