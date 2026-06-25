# Simulating real user interaction (user-event)

## Describe what the user does, not which events fire

`user-event` lets you express interactions at the level of *intent* — "the user types
their name," "the user clicks Submit" — instead of dispatching individual low-level
DOM events. That framing is the point: a test that mirrors what a person does carries
more confidence than one that fires a single synthetic event the user would never
trigger in isolation.

## Why it beats a single synthetic event

A real action produces a *sequence* of events. Typing one character in a field isn't
one event — it's focus, then keydown, keypress/input, value update, keyup, and so on.
`user-event` replays that realistic chain, so any handler along the way runs exactly
as it would for a user. Dispatching a single named event by hand can skip steps a
real interaction would hit, letting a handler bug slip through (a false positive) or
forcing you to manually orchestrate events the library would produce for you.

## It enforces that the element is actually interactable

`user-event` adds **visibility and interactability checks**: it won't let you "click"
an element that's hidden or "type" into one that's disabled, because a user couldn't
either. This catches a whole class of bugs that a lower-level event API masks by
permitting impossible interactions — a test that clicks a button covered by an
overlay, or types into a disabled input, passes mechanically while the real UI is
broken. By refusing impossible actions, `user-event` keeps the test honest about what
a user can really do.

## Good / Bad in practice

```js
// ❌ Bad — one synthetic event; skips focus/keydown/keyup, may miss handlers,
//          and happily "types" into a field a real user couldn't reach
fireEvent.change(screen.getByLabelText(/search/i), { target: { value: 'cat' } })

// ✅ Good — set up once, then drive the real interaction sequence
const user = userEvent.setup()
await user.type(screen.getByLabelText(/search/i), 'cat')
await user.click(screen.getByRole('button', { name: /search/i }))
```

```js
// ❌ Bad — clicking an element a user couldn't actually activate passes silently
fireEvent.click(screen.getByRole('button', { name: /submit/i })) // even if disabled/hidden

// ✅ Good — user-event refuses impossible interactions, so the test stays honest
const user = userEvent.setup()
await user.click(screen.getByRole('button', { name: /submit/i }))
// throws if the button is disabled or not visible — surfacing the real bug
```

## Set up once per test — ideally in your custom render

Call `userEvent.setup()` once at the start of a test (before rendering) and reuse the
returned `user` for every interaction. Better still, do the `setup()` inside a
**custom render** so each test receives a ready `user` with no boilerplate — see
[test-maintainability.md](./test-maintainability.md#custom-render-wrap-providers-once-and-set-up-user-event-there).

```js
// ✅ Good — one setup, reused; or returned from a custom render
const { user } = render(<App />) // custom render did userEvent.setup() for you
await user.click(screen.getByRole('button', { name: /menu/i }))
await user.type(screen.getByRole('searchbox'), 'cat')
```

## Judgment, not mechanics

The deterministic part — "prefer `user-event` over `fireEvent`" — is enforced by the
linter (`prefer-user-event`), so don't spend prose arguing the swap. The judgment
this reference is about is *fidelity*: model the actual interaction a user performs
(including the small realistic steps — focus before type, hover before a tooltip,
the full click sequence) so that when the test passes, you know the behavior holds
for a real person, and when it fails, it's pointing at a real interaction problem.

## Further reading

- Testing Library — user-event intro: <https://testing-library.com/docs/user-event/intro/>
- Kent C. Dodds — Common Mistakes with React Testing Library (fireEvent vs userEvent): <https://kentcdodds.com/blog/common-mistakes-with-react-testing-library>
