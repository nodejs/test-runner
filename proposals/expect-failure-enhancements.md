# Feature proposal: `expectFailure` label & matcher

## Summary

Update the `expectFailure` option in `test()` to accept different types of values, enabling both **custom label** and **error matching**. This proposal integrates the requirements from [nodejs/node#61570](https://github.com/nodejs/node/issues/61570), ensuring consistency with `skip`/`todo` while adding robust validation capabilities.

## API & Behavior

The behavior of `expectFailure` is strictly determined by the type of value provided:

### String: Failure label

When a **non-empty** `string` is provided, it acts as an output label (reason) for that test-case (identical to `skip` and `todo` options).

```js
test('fails with a specific reason', {
  expectFailure: 'Bug #123: Feature not implemented yet'
}, () => {
  throw new Error('boom');
});
```
- **Behavior**: The test is expected to fail. The string is treated as a label/reason.
- **Validation**: None. It accepts _any_ error.
- **Output**: The reporter displays the string literally, prefixed by a TAP comment character (e.g., `# EXPECTED FAILURE Bug #123…`).

### Matcher: Error constructor, Error-like object, RegExp, or validation function

When an `Error` constructor, `Error`-like object, `RegExp`, or validation function is provided, it is treated as validation to match against, mirroring the behaviour of [`assert.throws`](https://nodejs.org/docs/latest-v25.x/api/assert.html#assertthrowsfn-error-message) (possibly leveraging it under the hood).

```js
test('fails with matching error (RegExp)', {
  expectFailure: /expected error message/
}, () => {
  throw new Error('this is the expected error message');
});

test('fails with matching error (Class)', {
  expectFailure: RangeError
}, () => {
  throw new RangeError('Index out of bounds');
});
```

### Configuration Object: Reason & Validation

When a **Plain Object** with specific properties (`match`, `label`) is provided, it allows specifying both a failure reason and validation logic simultaneously.

```js
test('fails with reason and specific error', {
  expectFailure: {
    label: 'Bug #123: Edge case behavior', // Reason
    match: /Index out of bounds/              // Validation
  }
}, () => {
  throw new RangeError('Index out of bounds');
});
```
- **Properties**:
    - `label` (String): The failure reason/label (displayed in reporter).
    - `match` (RegExp | Object | Function | Class): Validation logic. This is passed directly to `assert.throws` validation argument, supporting all its capabilities.
- **Requirement**: The object must contain at least one of `label` or `match`.
- **Behavior**: The test passes **only if** the error matches the `match` criteria.
- **Output**: The reporter displays the `label`.

### Equivalence

The following configurations are equivalent in behavior:

**1. Reason only:**
```js
expectFailure: 'reason';
```

**2. Validation only:**
```js
expectFailure: /error/
expectFailure: { match: /error/ }
```

**3. Catch-all (Any Error):**
```js
expectFailure: true
```

## Ambiguity Resolution
Potential ambiguity between a **Matcher Object** and a **Configuration Object** is resolved as follows:

1.  **String** → Reason.
2.  **RegExp** or **Function** → Matcher (Validation).
3.  **Object**:
    *   **Empty Object** (`{}`) → **Error**: throws `ERR_INVALID_ARG_VALUE`.
        ```js
        // Uses Node.js standard error code
        throw new ERR_INVALID_ARG_VALUE(
          'expectFailure',
          expectFailure,
          'must not be an empty object'
        );
        ```
    *   If the object contains **only** `match` and/or `label` properties → **Configuration Object**.
    *   Otherwise → **Matcher Object** (passed to `assert.throws` for property matching).

## Activation & Truthiness

To maintain strict consistency with `todo` and `skip` options:
*   The feature is **disabled** only if `expectFailure` is `undefined` or `false`.
*   **All other values** enable the feature (treat as truthy).
    *   `expectFailure: ''` (Empty String) → **Enabled** (treats as generic failure expectation).
    *   `expectFailure: 0` → **Enabled** (treated as a Matcher Object unless specific logic excludes numbers, but per consistency it enables the feature).

### Flat Options (`expectFailureError`)

It was proposed to split the options into `expectFailure` (reason) and `expectFailureError` (validation).
This was rejected in favor of the nested/polymorphic structure using `match` and `label` properties. This syntax was selected as the preferred choice for its readability and clarity:
* `match`: Clearly indicates "fails **matching** this error" (Validation).
* `label`: Clearly indicates the **label** or reason for the expected failure.
This approach keeps related configuration grouped without polluting the top-level options namespace.

## Implementation Details

### Validation Logic

The implementation leverages `assert.throws` internally to perform error validation.
- If `expectFailure` is a Matcher (RegExp, Class, Object), it is passed as the second argument to `assert.throws(fn, expectFailure)`.
- If `expectFailure` is a Configuration Object, `expectFailure.match` is passed to `assert.throws`.
