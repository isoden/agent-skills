# Avoiding implementation details

This is the single highest-leverage judgment in frontend testing, and the one a
linter can never make for you, because "is this an implementation detail?" depends on
what the software is *supposed to do*.

## What counts as an implementation detail

An implementation detail is anything the users of your code never use, see, or know
about. Anchor on the two real users:

- the **end user**, who sees and interacts with the rendered output; and
- the **developer user**, who consumes the component's public API (props, return
  values).

Anything outside those two — internal state variable names, private methods,
component instance internals, lifecycle specifics, which handler is wired where — is
an implementation detail. Testing it invents a third **"test user"** that you have to
keep happy even though nobody ever ships that user.

## Why coupling to internals hurts — in both directions

Tests bound to implementation details fail you twice:

- **False negatives (brittle tests).** A behavior-preserving refactor breaks the
  test. Rename an internal state field, restructure a component, swap a hook — the UI
  is identical, the user is unaffected, yet the suite goes red. That's pure
  maintenance cost with no bug caught. A useful litmus test: *if a refactor that
  changes no behavior forces you to edit the test, the test was brittle.*
- **False positives (false confidence).** The test stays green while the app is
  actually broken. If a test only checks an internal state value, you can break the
  wiring that turns that state into visible behavior and the test never notices. The
  confidence is illusory — the worst outcome, because you trust a suite that lies.

Testing through the user-facing surface avoids both: behavior-preserving refactors
keep passing (no false negatives), and broken behavior turns the test red (no false
positives).

## Write resilient tests by querying the way users perceive

Brittleness usually enters through the *selectors*. Querying by CSS class or element
index couples the test to structure that exists for other reasons. The fix is the
query priority order (see [queries.md](./queries.md)): find elements by role, label,
and visible text — the things a user actually perceives — and treat `data-testid` as
a deliberate last resort. A concrete tell: if adding a second matching element forces
you to change `querySelector('.btn')` into `querySelectorAll('.btn')[1]`, the test
was coupled to layout, not behavior.

## Good / Bad in practice

Consider a counter `<Counter />` that renders a button showing the current count.

```js
// ❌ Bad — asserts on internal state. Breaks when you rename `count` to `value`
//          (false negative), and stays green if the button stops rendering the
//          state (false positive). Tests a "test user" nobody ships.
const { result } = renderHook(() => useCounter())
act(() => result.current.increment())
expect(result.current.count).toBe(1)
```

```js
// ✅ Good — asserts on what the user sees and does. Survives internal refactors,
//          fails if the visible behavior actually breaks.
const user = userEvent.setup()
render(<Counter />)
const button = screen.getByRole('button', { name: /count is 0/i })
await user.click(button)
expect(button).toHaveTextContent(/count is 1/i)
```

> Note: testing a genuinely public custom hook (a *developer user* API) is
> legitimate — that's the API consumers rely on. The anti-pattern is reaching into a
> component's *private* internals that no real user touches. The two real users (end
> user, developer user) are the line; everything else is a test user.

## Don't rely on willpower — let tooling steer you

It's tempting to say "I'll just be disciplined and avoid testing internals." That
doesn't scale. Use tools that make user-centric testing the path of least resistance:
Testing Library deliberately gives you no easy handle on component internals, so the
easy way *is* the right way. Choosing the right library removes the temptation rather
than asking you to resist it.

## Quick check before committing a test

- Could a teammate refactor the component's internals without touching this test? If
  not, it's coupled to implementation.
- If the feature silently broke for users, would this test fail? If not, it's giving
  false confidence.
- Is every query something a human tester could follow (role/label/text)? If you're
  reaching into the DOM, reconsider.

## Further reading

- Kent C. Dodds — Testing Implementation Details: <https://kentcdodds.com/blog/testing-implementation-details>
- Kent C. Dodds — Avoid the Test User: <https://kentcdodds.com/blog/avoid-the-test-user>
- Kent C. Dodds — Making Your UI Tests Resilient to Change: <https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change>
