# Feature proposal: `flaky`

This flag causes a test-case or suite to be re-run a number of times until it either passes or has not passed after the final re-try.

## API

```js
describe.flaky('some temperamental thing', () => {
  it('may take several times', () => {…});
  
  it('may also take several times', () => {…});
});

it.flaky('should do something', () => {…});
```

```js
describe('some temperamental thing', { flaky: true }, () => {
  it('may take several times', () => {…});
  
  it('may also take several times', () => {…});
});

it('may take several times', { flaky: true }, () => {…});
it('may also take several times', { flaky: 100 }, () => {…});
```

When `flaky` is `true`, the test harness re-tries the suite or test-case up to the default number of times (eg 20), inclusive.

When `flaky` is falsy (the default), the test harness does not re-try the suite or test-case.

When `flaky` is a positive integer, the test harness re-tries the suite or test-case up to the specified number of times, inclusive.

When `flaky` is a non-positive integers or non-boolean, the test harness emits/throws an error.

When both the suite and an included test-case specify the `flaky` flag, the test-case's `flaky` value wins.

## Output

When a suite or test-case is marked `flaky`, the number of re-tries taken to achieve a passing result is reported with that suite or test-case.

The summary also lists the total number of _test-cases_ marked flaky; a "flaky" suite containing 10 test-cases increases the total flaky count by 10.

### Example: TAP

```
ok 1 - may take several times # FLAKY 42 re-tries

tests 1
suites 1
pass 1
fail 0
flaky 1
cancelled 0
skipped 0
todo 0
```

## Notes

Feature documentation (possibly in a Learn article rather than API documentation) warns users of common caveats related to re-tries (e.g. [idempotence](https://en.wikipedia.org/wiki/Idempotence), statefulness).
