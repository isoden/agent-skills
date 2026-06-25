# Mocking: every mock trades away confidence

## The trade-off, stated plainly

The guiding principle has a direct corollary for mocks: since "the more your tests
resemble the way your software is used, the more confidence they can give you," and a
mock **severs a real-world connection**, *every mock you add reduces confidence*. That
doesn't make mocking wrong — it makes it a deliberate trade you should only take when
the real thing is genuinely impractical, costly, or impossible.

Good reasons to mock:

- The real dependency has unacceptable side effects (don't charge a real credit card
  in a checkout test).
- It's slow, flaky, or out of your control in a way that would make the test
  unreliable (third-party network).
- It's literally unavailable in the test environment.

A **bad** reason to mock: **performance**. Shaving milliseconds by stubbing a
component is a poor trade — you give up real confidence to save time you didn't need
to save. If rendering is genuinely slow, that's a real performance bug to fix, not to
paper over with a mock.

Rule of thumb for UI tests: mock the **network and animations**; keep most of your own
production code real. The higher up the testing trophy you are (integration → E2E),
the *less* you should mock.

```js
// ❌ Bad — mocking your own component to "simplify" or speed up the test.
//          Now the test can't prove the real pieces work together.
jest.mock('./PriceSummary')

// ✅ Good — render the real component tree; only the network is faked (see below).
render(<Checkout />)
```

## Stop mocking `fetch` — intercept at the network layer

Mocking `fetch` (or your API client) directly is brittle: you end up
**re-implementing backend behavior inside test files**, duplicating that fake across
every test, and losing confidence that the real request/response wiring works.

Prefer intercepting at the **network layer** with
[MSW (Mock Service Worker)](https://mswjs.io/): define request handlers once and reuse
them across tests *and* local development.

Why this is better than a `fetch` mock:

- **It catches real bugs.** If you get the request wrong (wrong URL, method, or body),
  no handler matches, so the request fails — and the test **correctly fails**. A
  direct `fetch` mock would happily return your canned value and hide the mistake.
- **No duplicated fake.** Handlers live in one place; tests don't each re-describe the
  backend.
- **Edge cases without setup churn.** Override a handler per-test to simulate a 500 or
  an empty result, without rebuilding the whole mock.
- **Refactor-friendly.** Tests track the user-visible experience, not the
  implementation of how data is fetched.

```js
// ❌ Bad — re-implements the backend in the test; wrong request still "passes"
global.fetch = jest.fn(() =>
  Promise.resolve({ json: () => Promise.resolve({ user: { name: 'Ada' } }) }),
)

// ✅ Good — MSW intercepts the real request at the network boundary
import { setupServer } from 'msw/node'
import { http, HttpResponse } from 'msw'

const server = setupServer(
  http.get('/api/user', () => HttpResponse.json({ name: 'Ada' })),
)
beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

test('greets the signed-in user', async () => {
  render(<Profile />)
  expect(await screen.findByText(/welcome, ada/i)).toBeInTheDocument()
})

// per-test edge case — no rebuild, just override the handler
test('shows an error when the profile fails to load', async () => {
  server.use(http.get('/api/user', () => new HttpResponse(null, { status: 500 })))
  render(<Profile />)
  expect(await screen.findByRole('alert')).toHaveTextContent(/unable to load/i)
})
```

## A note for the linter boundary

"Don't assign a `jest.fn()`/`vi.fn()` to `global.fetch`" is close to a deterministic
rule a custom lint could flag, but *which* boundary to mock and *when* a mock is worth
its confidence cost are judgment calls — which is why they live here in prose.

## Further reading

- Kent C. Dodds — The Merits of Mocking: <https://kentcdodds.com/blog/the-merits-of-mocking>
- Kent C. Dodds — Stop Mocking Fetch: <https://kentcdodds.com/blog/stop-mocking-fetch>
- Mock Service Worker: <https://mswjs.io/>
