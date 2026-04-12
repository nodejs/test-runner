# Support Multiple Snapshot Formats in the Node.js Test Runner

The current snapshot implementation in the Node.js Test Runner assumes a single snapshot file format (`.snapshot`).

However, Node.js itself natively supports multiple module and data formats:

* `.cjs` (CommonJS)
* `.mjs` (ESM)
* `.js` (Both)
* `.json`

The test runner already embraces this flexibility for **test files themselves** (it automatically discovers `.js`, `.cjs`, and `.mjs` test files), so one would expect snapshots to be loadable the same way.

## Goal

When resolving a snapshot file, the test runner should support multiple extensions.

Example resolution order:

```text
<test>.snapshot.mjs
<test>.snapshot.cjs
<test>.snapshot.js
<test>.snapshot.json
```

Example error:

```
Multiple snapshot files found for test "foo.test.js":
- foo.test.snapshot.mjs
- foo.test.snapshot.json
```
