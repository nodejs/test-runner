# Feature proposal: `expectFailure` enhancements

The `expectFailure` option currently supports a boolean value to indicate that a test is expected to fail. This proposal aims to expand `expectFailure` to support validation of the failure (matching the error) and providing a custom message (reason), aligning it with `skip` and `todo` while adding `assert.throws`-like capabilities.

## API

```js
// 1. String as a message (Reason) - Aligns with skip/todo behavior
test('fails with specific reason', {
  expectFailure: 'Bug #123: Feature not implemented yet'
}, () => {
  throw new Error('boom');
});

// 2. RegExp as a matcher (Validation)
test('fails with matching error', {
  expectFailure: /boom/
}, () => {
  throw new Error('boom');
});

// 3. Object for explicit control (Both)
test('fails with reason and validation', {
  expectFailure: {
    message: 'Bug #123', // Displayed in reporter
    match: /boom/      // Validation logic (RegExp, Object, Function, or String)
  }
}, () => {
  throw new Error('boom');
});
```

## Behavior

### String Value (`expectFailure: 'msg'`)
Treats the string as a **reason** for the expected failure, similar to `skip: 'msg'` or `todo: 'msg'`.
- **Validation**: None (any error is accepted).
- **Output**: The string is displayed in the reporter output (e.g., TAP directive `# EXPECTED FAILURE msg`).
- **Rationale**: Preserves consistency with other `test()` options where a string implies a reason/description.

### RegExp Value (`expectFailure: /regex/`)
Treats the regex as a **matcher** for the error thrown.
- **Validation**: The error message must match the regex. If it doesn't match (or no error is thrown), the test fails.
- **Output**: Standard expected failure output.

### Object Value (`expectFailure: { ... }`)
Allows combining a reason message with validation logic.
- `message`: String. Displayed in reporter output as the reason.
- `match`: `RegExp | Function | Object | String`. Validates the error using logic similar to `assert.throws(fn, match)`.
    - If `match` is a string, it validates that the error message includes the string (or exact match, depending on `assert.throws` semantics).

## Ambiguity Resolution
There is a potential conflict if users expect `expectFailure: 'string'` to match the error message (like `assert.throws`) rather than be the reason.

**Proposed Decision**: Consistency with `test()` options `skip` and `todo` takes precedence. Therefore, a top-level string is a **message/reason**. Users wanting to match a specific string error should use `{ match: 'string' }` or a RegExp `/string/`.

## Output Example (TAP)

```tap
# String (Reason)
ok 1 - fails with specific reason # EXPECTED FAILURE Bug #123: Feature not implemented yet

# RegExp (Validation)
ok 2 - fails with matching error # EXPECTED FAILURE

# Object (Both)
ok 3 - fails with reason and validation # EXPECTED FAILURE Bug #123
```
