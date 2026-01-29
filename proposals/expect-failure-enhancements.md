# Feature proposal: `expectFailure` enhancements

## Summary
Update the `expectFailure` option in `test()` to accept different types of values, enabling both **custom failure messages** and **error validation**. This proposal integrates the requirements from [nodejs/node#61570](https://github.com/nodejs/node/issues/61570), ensuring consistency with `skip`/`todo` while adding robust validation capabilities.

## API & Behavior

The behavior of `expectFailure` is strictly determined by the type of value provided:

### 1. String: Failure Reason
When a **non-empty string** is provided, it acts as a documentation message (reason), identical to `skip` and `todo` options.

```js
test('fails with a specific reason', {
  expectFailure: 'Bug #123: Feature not implemented yet'
}, () => {
  throw new Error('boom');
});
```
- **Behavior**: The test is expected to fail. The string is treated as a label/reason.
- **Validation**: None. It accepts *any* error.
- **Output**: The reporter displays the string (e.g., `# EXPECTED FAILURE Bug #123...`).

### 2. RegExp: Error Matcher
When a **RegExp** is provided, it acts as a validator for the thrown error.

```js
test('fails with matching error', {
  expectFailure: /expected error message/
}, () => {
  throw new Error('this is the expected error message');
});
```
- **Behavior**: The test passes **only if** the thrown error matches the regular expression.
- **Validation**: Strict matching against the error message.
- **Output**: Standard expected failure output.

### 3. Object: Reason & Validation
When an **Object** is provided, it allows specifying both a failure reason and validation logic simultaneously.

```js
test('fails with reason and specific error', {
  expectFailure: {
    message: 'Bug #123: Edge case behavior', // Reason
    match: /Index out of bounds/             // Validation
  }
}, () => {
  throw new RangeError('Index out of bounds');
});
```
- **Properties**:
    - `message` (String): The failure reason/label (displayed in reporter).
    - `match` (RegExp | Object | Function): Validation logic (similar to `assert.throws` validation argument).
- **Behavior**: The test passes **only if** the error matches the `match` criteria.
- **Output**: The reporter displays the `message`.

## Ambiguity Resolution
Potential ambiguity is resolved by strict type separation:
*   `typeof value === 'string'` → **Reason**
*   `value instanceof RegExp` → **Matcher**
*   `typeof value === 'object'` → **Both** (Explicit properties)

## Edge Cases & Implementation Details

### Empty String (`expectFailure: ''`)
Following standard JavaScript truthiness rules, an empty string should be treated as **falsy**.
*   `expectFailure: ''` behaves exactly like `expectFailure: false`.
*   The feature is **disabled**, and the test is expected to pass normally.

### Type Safety for `this.passed`
The implementation must ensure that `this.passed` remains a strict `boolean`.
Assigning a string directly (e.g., `this.passed = this.expectFailure`) is unsafe as it introduces type pollution.

**Recommended Implementation Logic:**
```javascript
// When an error is caught:
this.passed = !!this.expectFailure; // Forces conversion to boolean
```
*   If `expectFailure` is `"reason"` → `true` (Test Passes)
*   If `expectFailure` is `""` → `false` (Test Fails, as expected failure was not active)
