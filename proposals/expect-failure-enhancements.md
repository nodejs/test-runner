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

### 2. Matcher: RegExp, Class, or Error Object
When a **RegExp**, **Class** (Function), or **Error Object** is provided directly, it acts as the validation logic. This leverages `assert.throws` behavior directly.

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

### 3. Configuration Object: Reason & Validation
When a **Plain Object** with specific properties (`with`, `message`) is provided, it allows specifying both a failure reason and validation logic simultaneously.

```js
test('fails with reason and specific error', {
  expectFailure: {
    message: 'Bug #123: Edge case behavior', // Reason
    with: /Index out of bounds/              // Validation
  }
}, () => {
  throw new RangeError('Index out of bounds');
});
```
- **Properties**:
    - `message` (String): The failure reason/label (displayed in reporter).
    - `with` (RegExp | Object | Function | Class): Validation logic. This is passed directly to `assert.throws` validation argument, supporting all its capabilities.
- **Behavior**: The test passes **only if** the error matches the `with` criteria.
- **Output**: The reporter displays the `message`.

### Equivalence
The following configurations are equivalent in behavior:

**1. Reason only:**
```js
expectFailure: 'reason'
expectFailure: { message: 'reason' }
```

**2. Validation only:**
```js
expectFailure: /error/
expectFailure: { with: /error/ }
```

**3. Catch-all (Any Error):**
```js
expectFailure: true
expectFailure: {}
```

## Ambiguity Resolution
Potential ambiguity between a **Matcher Object** and a **Configuration Object** is resolved as follows:

1.  **String** → Reason.
2.  **RegExp** or **Function** → Matcher (Validation).
3.  **Object**:
    *   If the object contains `with` or `message` properties → **Configuration Object**.
    *   Otherwise → **Matcher Object** (passed to `assert.throws` for property matching).

## Activation & Truthiness
To maintain strict consistency with `todo` and `skip` options:
*   The feature is **disabled** only if `expectFailure` is `undefined` or `false`.
*   **All other values** enable the feature (treat as truthy).
    *   `expectFailure: ''` (Empty String) → **Enabled** (treats as generic failure expectation).
    *   `expectFailure: 0` → **Enabled** (treated as a Matcher Object unless specific logic excludes numbers, but per consistency it enables the feature).

### Flat Options (`expectFailureError`)
It was proposed to split the options into `expectFailure` (reason) and `expectFailureError` (validation).
This was rejected in favor of the nested/polymorphic structure using `with` and `message` properties. This syntax was selected as the preferred choice for its readability and clarity:
*   `with`: Clearly indicates "fails **with** this error" (Validation).
*   `message`: Clearly indicates the **reason** or label for the expected failure.
This approach keeps related configuration grouped without polluting the top-level options namespace.

## Implementation Details

### Validation Logic
The implementation leverages `assert.throws` internally to perform error validation.
- If `expectFailure` is a Matcher (RegExp, Class, Object), it is passed as the second argument to `assert.throws(fn, expectFailure)`.
- If `expectFailure` is a Configuration Object, `expectFailure.with` is passed to `assert.throws`.