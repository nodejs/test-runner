# Suite-level hooks

## Problem(s) to solve

* Hooks (`after`/`afterEach`, `before`/`beforeEach`) are currently global, which causes them to trigger for subtests. That is often counter-productive, rendering subtests useless (they end up clobbering each other).
* Tests often need common setup but are affected by concurrency (for instance, mocks). In those scenarios, either concurrency must be disabled or a lot of code must be repeated.
  * For mocks, it is generally desirable to reset them between tests.

## API & Behaviour

Note in the below:

* `s` is `SuiteContext`
* `th` is `TestHookContext`
* `t` is `TestContext`

```ts
type SuiteContextHook = (
  c?: (th: TestHookContext): void,
  options: HookOptions,
);
type SuiteContext = {
  // …
  after: SuiteContextHook,
  afterEach: SuiteContextHook,
  before: SuiteContextHook,
  beforeEach: SuiteContextHook,
  mock: SuiteContextMockTracker → MockTracker,
}
```
```ts
type TestHookContext = Record<string | Symbol, any>;
```
```ts
type TestContext = {
  // …
  bikeshed: TestHookContext,
}
```

```js
describe('Suite-level hooks', (s) => {
  s.before((th) => {
    const foo = s.mock.fn();
    s.mock.module('foo', { exports: { default: foo } });
    const bar = s.mock.fn();
    s.mock.module('bar', { exports: { default: bar } });

    th.mocks = {
      bar: bar.mock,
      foo: foo.mock,
    };
  });

  it('should abort on error', (t) => {
    t.bikeshed.mocks.foo.mockImplementation(() => false);

    widget();

    t.test('call foo', () => assert.equal(t.bikeshed.mocks.foo.callCount(), 1));
    t.test('call bar', () => assert.equal(t.bikeshed.mocks.bar.callCount(), 0));
  });

  it('should succeed on happy-path', (t) => {
    widget();

    t.test('call foo', () => assert.equal(t.bikeshed.mocks.foo.callCount(), 1));
    t.test('call bar', () => assert.equal(t.bikeshed.mocks.bar.callCount(), 1));
  });
});
```

A lot of subtle things are happening in the above:

Each test (`it`) receives a clone of `TestHookContext` (nested on a known/reserved slot currently called `bikeshed` because I couldn't think of a good name). This means each mock is distinct:
* The `mockImplementation` from _'should abort on error'_ applies to only its own copy of the mock of `foo` and does NOT affect the `foo` mock in _'should succeed on happy-path'_.
* A mock's calls (counters, arguments, etc) are isolated to the test.
* Mutations to the `TestHookContext` a test receives do not affect `TestHookContext` in other tests.

| | between tests (`it`) | between subtests (`t.test`)
:-- | :--: | :--:
`SuiteContext.mock.resetCalls()` | ✅ | ❌
`SuiteHook`s | ✅ | ❌

## Under the hood

The `SuiteContextHookFn` passed to a `SuiteContextHook` is run for every test, receiving a fresh `TestHookContext` mapped to the test to which it will be provided.

Each `SuiteContextMockTracker` will need to intelligently apply appropriate mocks.
