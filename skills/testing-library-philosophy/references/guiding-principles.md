# Guiding principles: what to test, and how much

## Confidence is the actual goal

A test exists to give you confidence that the software works for the people who use
it. Testing Library's guiding principle frames the trade-off precisely: the closer a
test resembles real usage, the more confidence a passing run earns. Treat
"resemblance to real usage" as the lever you optimize — not coverage percentage, not
number of tests.

There are only two users your code actually has:

- the **end user**, who interacts with the rendered output (clicks, types, reads); and
- the **developer user**, who consumes the component's public API (props, returned
  values).

Write tests from one of those two perspectives. A test written from any other
vantage point — reaching into internal state, calling private methods — is serving a
phantom "test user" that ships to nobody.

## The Testing Trophy: spend effort where ROI is highest

Kent C. Dodds reframes the classic testing pyramid as a **Testing Trophy** to reflect
how modern tooling shifted the value of each layer. From bottom to top:

- **Static** — TypeScript and ESLint catch typos and type errors for almost no
  ongoing cost.
- **Unit** — isolated pieces; cheap and fast, but heavy mocking buys false
  confidence because it can't prove pieces work *together*.
- **Integration** — multiple units working together through the real user-facing
  surface. This is the **sweet spot**: high confidence without the maintenance and
  slowness of full end-to-end runs.
- **End-to-end** — a full workflow in a realistic environment; highest confidence,
  highest cost and slowest.

As you move up: confidence rises, but so does cost (slower to write, run, and
maintain). Integration tests sit where the curves cross most favorably, which is why
the guidance is **"write tests, not too many, mostly integration."** Invest the bulk
of your effort there.

## Decide what to test by use-case coverage, not code coverage

100% line coverage can still miss the scenarios users care about, and chasing it
pushes you toward brittle, isolated unit tests. Instead:

- For each meaningful path, ask **"what user need does this serve?"** rather than
  "which branch is untested?"
- Prioritize by **usage and cost of failure**: *what part of this app would upset
  users most if it broke?* Rank those with the team and test them first.
- A pragmatic order: start with happy-path **end-to-end** coverage of the highest-
  value journeys (disproportionate confidence, quickly), then add **integration**
  tests for edge cases, then **unit** tests only for genuinely complex logic.
  Accrete coverage over time instead of demanding a number up front. (This stays
  consistent with the Trophy: E2E covers only the few highest-value journeys, while
  integration carries the bulk of the suite.)

## Good / Bad in practice

```js
// ❌ Bad — heavily-mocked unit test. Asserts a child was called with props.
//          Can't prove the pieces actually work together; green even if the real
//          form never submits. Optimizes code coverage, not use-case coverage.
jest.mock('./SubmitButton')
render(<LoginForm onSubmit={onSubmit} />)
expect(SubmitButton).toHaveBeenCalledWith({ disabled: true }, expect.anything())

// ✅ Good — integration test of the real use case, through the user-facing surface.
//          Covers "a user can log in" end-to-end across the composed components.
const user = userEvent.setup()
render(<LoginForm onSubmit={onSubmit} />)
await user.type(screen.getByLabelText(/email/i), 'ada@example.com')
await user.type(screen.getByLabelText(/password/i), 'correct horse')
await user.click(screen.getByRole('button', { name: /log in/i }))
// onSubmit is the component's public callback prop — the developer-user's contract —
// so asserting on it is observable behavior, NOT an internal mock-call assertion.
expect(onSubmit).toHaveBeenCalledWith({
  email: 'ada@example.com',
  password: 'correct horse',
})
```

## Test isolation

Each test should run against its own fresh instance and never depend on another
test's execution, ordering, or side effects. Shared mutable state creates implicit
dependencies — removing, skipping, or reordering one test silently breaks others and
makes failures hard to diagnose. Render fresh per test (a small `renderX()` helper
keeps each test self-contained and explicit; auto-cleanup handles teardown), and
organize tests around user workflows rather than per-function fragments.

The old "one assertion per test" rule is obsolete: modern matchers report exactly
which assertion failed, so a cohesive multi-assertion test that walks a real user
scenario is fine — and reads as better documentation.

## Further reading

- Testing Library — Guiding Principles: <https://testing-library.com/docs/guiding-principles/>
- Kent C. Dodds — Write Tests. Not Too Many. Mostly Integration.: <https://kentcdodds.com/blog/write-tests>
- Kent C. Dodds — How to Know What to Test: <https://kentcdodds.com/blog/how-to-know-what-to-test>
- Kent C. Dodds — Test Isolation with React: <https://kentcdodds.com/blog/test-isolation-with-react>
