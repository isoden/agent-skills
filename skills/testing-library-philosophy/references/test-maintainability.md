# Test maintainability: AHA, abstraction, and test size

Test code optimizes for a *different* target than production code. The load-bearing
question for a test file is: **how quickly can a reader tell what makes two similar
tests different, and why that difference matters?** Every structural choice —
inline vs. extract, how many assertions per test, how much nesting — should be judged
against that question. This is pure judgment; a linter can flag a symptom (deep
nesting) but not the principle.

## AHA: Avoid Hasty Abstraction

Kent C. Dodds applies **AHA — "Avoid Hasty Abstraction"** to tests. Picture a
spectrum between two failure modes:

- **ANA ("Absolutely No Abstraction")** — everything inlined, nothing shared.
  Near-identical setup is copy-pasted across cases, and the one line that actually
  varies (a coordinate, an input flag) is buried in boilerplate noise. The reader
  can't see *what changes*.
- **Over-DRY** — every repeated line abstracted away via deep `describe`/`beforeEach`
  nesting, shared mutable variables hoisted to the top, and conditionals threaded
  into shared helpers. Now the reader must mentally execute the whole file to
  understand any single test. The abstraction *hides* the differences they're
  hunting for.

Aim for the **mindful middle**: abstract only when duplication is actively making the
differences hard to scan — not preemptively. For a two-test file, inlining wins;
the abstraction would cost more comprehension than it saves.

### The litmus test

> Abstract when tests have become hard to scan for their differences — and not
> before.

Default to inline. Extract only when (a) the pattern genuinely repeats, (b) the
repetition is creating real friction, and (c) there's enough volume to justify it.

## The `setup(overrides)` factory

The most useful test abstraction is a small **local factory** that builds inputs from
sensible defaults and accepts an overrides object, so each test passes only the
values it cares about. The meaningful difference between two tests collapses to one
or two visible lines.

```js
// ❌ Bad (ANA) — the one value that differs is lost in duplicated boilerplate
test('rejects a user under 18', () => {
  const user = { name: 'Sam', email: 's@x.com', country: 'US', age: 17 }
  expect(validate(user).errors).toContain('must be 18+')
})
test('accepts a user of 18', () => {
  const user = { name: 'Sam', email: 's@x.com', country: 'US', age: 18 }
  expect(validate(user).errors).toHaveLength(0)
})

// ✅ Good — defaults hidden, the distinguishing input is the only thing in view
function setup(overrides) {
  return { name: 'Sam', email: 's@x.com', country: 'US', age: 30, ...overrides }
}
test('rejects a user under 18', () => {
  expect(validate(setup({ age: 17 })).errors).toContain('must be 18+')
})
test('accepts a user of 18', () => {
  expect(validate(setup({ age: 18 })).errors).toHaveLength(0)
})
```

## The `renderFoo()` helper (React)

For a component test file with enough tests, a per-file render helper encapsulates the
`render(...)`, common queries, and interaction shortcuts — gated on volume, not
written for the first two tests.

```js
// ✅ Good — once the file has many tests, this removes repetition without hiding intent
function renderLoginForm(overrides) {
  const props = { onSubmit: jest.fn(), ...overrides }
  render(<LoginForm {...props} />)
  const user = userEvent.setup()
  return {
    props,
    user,
    email: screen.getByLabelText(/email/i),
    submit: screen.getByRole('button', { name: /log in/i }),
  }
}
```

## Custom render: wrap providers once, and set up user-event there

Real apps render inside providers — theme, router, i18n, a store. Repeating that
wrapper in every test is noise, and forgetting it makes tests fail for the wrong
reason. The canonical fix from the React Testing Library setup docs is a **custom
render**: a module that wraps `render` with your app's providers and **re-exports
everything** from RTL, so tests import from your module instead of the library.

This is the one abstraction that's justified **globally** rather than colocated (see
the next section): every test needs the *same* stable provider shell, so a shared
`test-utils` module doesn't accumulate per-test conditionals the way a data-setup
helper would.

The setup docs also recommend doing **user-event's `setup()` inside the custom
render** and returning the `user` instance, so each test gets a correctly-configured
user without an extra boilerplate line. (Call `userEvent.setup()` *before* rendering.)

```js
// test-utils.jsx — wrap providers + set up user-event in one place
import { render } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { ThemeProvider } from '../theme'
import { RouterProvider } from '../router'

function AllProviders({ children }) {
  return (
    <ThemeProvider>
      <RouterProvider>{children}</RouterProvider>
    </ThemeProvider>
  )
}

// returns { user, ...renderResult } so tests never re-setup either concern
function renderWithProviders(ui, options) {
  return {
    user: userEvent.setup(),
    ...render(ui, { wrapper: AllProviders, ...options }),
  }
}

export * from '@testing-library/react' // re-export so tests import from here
export { renderWithProviders as render } // override render with the wrapped one
```

```js
// a test — no provider boilerplate, no per-test userEvent.setup()
import { render, screen } from '../test-utils'

test('a user can open the menu', async () => {
  const { user } = render(<App />)
  await user.click(screen.getByRole('button', { name: /menu/i }))
  expect(screen.getByRole('navigation')).toBeVisible()
})
```

Keep the provider list in the custom render aligned with production. A test that
renders under different providers than the real app is, quietly, testing a different
application.

## Keep abstractions colocated

Keep your **data/setup helpers** (the `setup(overrides)` factories, scenario builders,
per-feature `renderFoo`) **local to the file** that needs them, not hoisted into a
distant shared module. A far-away shared helper accumulates conditionals as each new
caller bends it to its needs — which is exactly how DRY test helpers rot. A helper
that lives next to its tests is easy to read, change, and reason about.

The exception is the stable provider shell above: the custom render belongs in a
shared `test-utils` module precisely because it *doesn't* vary per test. The
distinction is volatility — share what's stable for everyone, colocate what's specific
to a scenario.

## Parameterized tests: the one place full DRY fits

For a pure function with many input→output cases, the logic is identical and only the
data varies — so a table of cases is both DRY *and* maximally scannable. Adding a case
is adding a row.

```js
// ✅ Good — data table; each row is one self-evident case
test.each`
  a    | b    | expected
  ${1} | ${1} | ${2}
  ${2} | ${3} | ${5}
  ${-1}| ${1} | ${0}
`('add($a, $b) === $expected', ({ a, b, expected }) => {
  expect(add(a, b)).toBe(expected)
})
```

## Write fewer, longer tests

A corollary of resembling real usage: prefer **fewer, longer integration-style tests**
that walk a whole user workflow over many tiny single-assertion tests. The old "one
assertion per test" rule is outdated orthodoxy from frameworks with poor failure
output; modern runners report exactly which assertion failed, so a cohesive scenario
test is fine — and reads as better documentation.

Think of a **workflow a manual tester would follow** and put every step in one test.
Over-splitting actively causes problems: shared mutable state between fragments, lost
isolation, and spurious async/act warnings. Keep **one Arrange (setup) per test**, but
allow multiple **Act → Assert** steps within it.

```js
// ❌ Bad — fragmented; shared state across tests, brittle, no real workflow proven
let result
test('renders empty cart', () => { result = render(<Cart />); /* ... */ })
test('adds an item', () => { /* depends on previous render */ })
test('shows the total', () => { /* depends on previous add */ })

// ✅ Good — one workflow, isolated, multiple act→assert steps
test('a user can add items and see the running total', async () => {
  const user = userEvent.setup()
  render(<Cart />)
  expect(screen.getByText(/your cart is empty/i)).toBeInTheDocument()

  await user.click(screen.getByRole('button', { name: /add apple/i }))
  expect(screen.getByText(/total: \$1\.00/i)).toBeInTheDocument()

  await user.click(screen.getByRole('button', { name: /add apple/i }))
  expect(screen.getByText(/total: \$2\.00/i)).toBeInTheDocument()
})
```

(This complements the isolation guidance in
[guiding-principles.md](./guiding-principles.md): "longer" never means "shared state
between tests" — each test still arranges its own world.)

## Further reading

- Kent C. Dodds — AHA Testing: <https://kentcdodds.com/blog/aha-testing>
- Kent C. Dodds — Write Fewer, Longer Tests: <https://kentcdodds.com/blog/write-fewer-longer-tests>
