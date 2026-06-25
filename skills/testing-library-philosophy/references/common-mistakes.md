# Common mistakes — the judgment-level ones

Many well-known Testing Library mistakes are mechanical and belong to the linter
(`screen` usage, naming the render result `wrapper`, manual `cleanup()`, awaiting
async queries, `findBy` vs `waitFor`+`getBy`, redundant `act()`). Those are in
[eslint-plugin.md](./eslint-plugin.md) — don't spend review effort on them.

What follows are the mistakes that require **judgment**: a linter can sometimes flag
the syntax, but deciding whether it's actually wrong depends on intent and on the UI.

## Querying in ways users can't

- **Skipping `*ByRole` when it fits.** Role queries match how users and screen
  readers perceive the UI, tolerate text split across elements, and give better
  failure output. Falling back to lower-priority queries without reason hides missing
  semantics. (Why the order exists → [queries.md](./queries.md).)
- **Test IDs instead of visible text.** Querying by `data-testid` when the visible
  text would work skips verification of the content (and i18n) users actually read,
  hiding regressions that *should* break the test. Use text; keep test IDs as a last
  resort.
- **Redundant or incorrect ARIA.** Semantic HTML already carries implicit roles. Add
  ARIA only to augment what semantics can't express — bolting on roles that duplicate
  or fight the native ones makes both the app and its tests worse.

## Asserting the wrong thing

- **Matching on raw DOM properties instead of intent-revealing matchers.** Asserting
  `button.disabled === true` instead of `toBeDisabled()` works but reads as
  mechanical poking and produces an opaque failure. Choose the matcher that names the
  user-facing behavior (→ [jest-dom.md](./jest-dom.md)).
- **`queryBy*` for positive existence.** `queryBy*` is for asserting absence; for
  presence use `getBy*`, whose throw includes a helpful DOM dump.

## Interacting unrealistically

- **`fireEvent` where a real interaction is multi-step.** A single synthetic event
  can skip the chain a real action produces, missing handler bugs. Model the actual
  interaction with `user-event` (→ [user-event.md](./user-event.md)).

## Misusing async waiting

These are about *what you put inside the wait*, which is a reasoning call:

- **Empty `waitFor` callback.** Waiting "a tick" with nothing asserted is fragile;
  the condition you're waiting for must live *inside* the callback so `waitFor` knows
  what success looks like.
- **Multiple assertions in one `waitFor`.** If an early assertion passes and a later
  one fails, the block keeps retrying until the full timeout — slow and confusing.
  Keep the awaited condition focused.
- **Side effects inside `waitFor`.** The callback runs repeatedly and at
  non-deterministic times, so firing events or other side effects inside it misfires.
  Do the action once, then wait on its observable result.

```js
// ❌ Bad — empty callback (waits a tick, asserts nothing), a side effect inside the
//          retry loop, and a query that should just be a findBy
await waitFor(() => {})
await waitFor(() => {
  fireEvent.click(button)              // side effect re-runs on every retry
  expect(screen.getByText('Saved')).toBeInTheDocument()
})

// ✅ Good — act once, then await the single observable condition
const user = userEvent.setup()
await user.click(button)
expect(await screen.findByText('Saved')).toBeInTheDocument()
```

## Testing implementation details

The umbrella mistake behind most brittleness and false confidence. It's significant
enough to have its own reference → [avoiding-implementation-details.md](./avoiding-implementation-details.md).

## Further reading

- Kent C. Dodds — Common Mistakes with React Testing Library: <https://kentcdodds.com/blog/common-mistakes-with-react-testing-library>
- Testing Library — Async methods: <https://testing-library.com/docs/dom-testing-library/api-async/>
