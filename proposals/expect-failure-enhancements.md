# Feature proposal: `expectFailure` enhancements

## Summary
Update the `expectFailure` option in `test()` to accept different types of values, enabling both **custom failure messages** and **error validation**. This aligns with `skip` and `todo` options while adding capabilities similar to `assert.throws`.

## API & Behavior

The behavior of `expectFailure` is determined by the type of value provided:

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
- **Rationale**: Maintains consistency with existing `test` options where a string implies a reason/description.

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

## Ambiguity Resolution
Potential ambiguity between "String as Reason" and "String as Matcher" is resolved by strict type separation:
*   `typeof value === 'string'` → **Reason** (Documentation only)
*   `value instanceof RegExp` → **Matcher** (Validation)

Users needing to match a specific string error message should use a RegExp (e.g., `/Error message/`) to avoid confusion.

## Edge Cases & Implementation Details

### Empty String (`expectFailure: ''`)
Following standard JavaScript truthiness rules, an empty string should be treated as **falsy**.

*   `expectFailure: ''` behaves exactly like `expectFailure: false`.
*   The feature is **disabled**, and the test is expected to pass normally.

### Type Safety for `this.passed`
The implementation must ensure that `this.passed` remains a strict `boolean`.
Assigning a string directly (e.g., `this.passed = this.expectFailure`) is unsafe as it introduces type pollution (assigning `""` or `"reason"` instead of `false`/`true`).

**Recommended Implementation Logic:**
```javascript
// When an error is caught:
this.passed = !!this.expectFailure; // Forces conversion to boolean
```
*   If `expectFailure` is `"reason"` → `true` (Test Passes)
*   If `expectFailure` is `""` → `false` (Test Fails, as expected failure was not active)
