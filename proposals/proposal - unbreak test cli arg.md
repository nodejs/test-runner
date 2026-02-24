# Proposal: un/break `--test`

Currently, this flag's behaviour is very unexpected, often resulting in silent-failure footguns, and precludes command nesting, as demonstrated in a very simple and typical CI setup:

```yaml title="unit & e2e tests"
- name: tests with coverage
  run: node --run test -- --test-reporter lcov
```

That results in

```sh
node --test --test-reporter lcov
```

In most cases, `--test-reporter` gets lost (the `lcov` reporter is not enabled); in a worse case, this unexpectedly causes tests within a directory named `lcov` to be run.

## More info

Currently (since always):

`--test` receives everything after it (space-delimited).

`--test` currently does 2 things:

* enables the test runner
* accepts paths for the runner to consume

This is similar to another existing feature with which we want to improve interop: `watch` mode.

`--watch` currently does 2 things:

* enables `watch` mode
* optionally accepts 1 value to override the entrypoint

`--watch-path` sets additional paths to include as well as the entrypoint.

```sh
node
  --watch
  --watch-path ./src/**/*.ts
  --watch-path ./test/**/*.ts
```

## Proposed options

### Option: n-number of `--test`s + `--watch-path`

`--test` is no-longer positional, accepting 1 value per occurrance of the flag. Supplying any value will override the default glob path.

When combined with test mode, `--watch`'s value is ignored (treated as an "on" flag)

`--watch-path` supplies additional paths to trigger the test-runner to re-run.

<table>
  <thead>
    <tr>
      <th>Description</th>
      <th>Code sample</th>
      <th>Resulting <code>--test</code> value</th>
    </tr>
  </thead>

<tbody>

<tr>
<td>
Test & watch modes enabled with defaults
</td>
<td>

```sh
node
  --test
  --watch
```

</td>
<td>

`**/*.test.{cjs,cts,mjs,mts,js,ts}`, [etc](https://nodejs.org/api/test.html#running-tests-from-the-command-line)

</td>
</tr>

<tr>
<td>
Test & watch modes enabled, overriding default
</td>
<td>

```sh
node
  --test ./src/foo/*.test.js
  --watch
```

</td>
<td>

`./src/foo/*.test.js`

</td>
</tr>
</tr>

<tr>
<td>
Test & watch modes enabled, using default & an additional path
</td>
<td>

```sh
node
  --test
  --test ./src/foo/*.test.js
  --watch
```

</td>
<td>

`./src/foo/*.test.js`
`**/*.test.{cjs,cts,mjs,mts,js,ts}`, [etc](https://nodejs.org/api/test.html#running-tests-from-the-command-line)

</td>
</tr>

<tr>
<td>
Test & watch modes enabled, multiple test paths & additional watch paths
</td>
<td>

```sh
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  --watch
  --watch-path ./src/quz/fixt.json
```

</td>
<td>

`./src/foo/*.test.js`
`./src/bar/*.test.js`

In addition to changes within the graph of those test, tests will also re-run if `./src/quz/fixt.json` changes.

</td>
</tr>

</tbody>
</table>

### Option: Break all the things

Currently `watch` has 2 flags (which are redundant):

```sh title='reads "main" etc from package.json'
node --watch
```
```sh
node --watch ./src/not-pjson-main.js
```
```sh
node
  --watch ./src/not-pjson-main.js
  --watch-path ./src/**/*.js
```

This could be simplified into just 1 `--watch` flag (where the default value is a single-element array of the derived entrypoint).

```sh title='reads "main" etc from package.json'
node --watch
```
```sh title='override derived entrypoint'
node --watch ./src/not-pjson-main.js
```
```sh title='package.json "main" + additional paths'
node
  --watch
  --watch ./src/**/*.js
  --watch ./src/not-pjson-main.js
```

```sh title='use default'
node --test
```
```sh title='override default with one or more paths'
node --test ./src/bar/*.test.js
```
```sh title='use default with one or more additional paths'
node
  --test
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
```

#### All together

`--watch` receives a clone of `--test`'s resolved value(s) **plus** `--watch`'s own values—with 1 exception: when both `test` and `watch` modes are enabled, `--watch` does not include a default entrypoint (it's irrelevant and it would likely result in perf waste at best, and unexpected behaviour at worst).

All examples are sequence-independent (all within the same heading behave the same).

##### Test & watch mode enabled:

```sh
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  --watch
```

##### Test & watch mode enabled with _additional_ watch path:

```sh
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  --watch ./src/something-else.js
```

### Option: Apply `--watch`'s current design to `--test`

`--test` is optional and optionally accepts 1 value, whose value defaults to node's built-in glob defaults (`./src/**/*.test.(c|m)?(j|t)s`, etc). Additional paths are supplied via a new `--test-path` flag:

<table>
  <thead>
    <tr>
      <th>Description</th>
      <th>Code sample</th>
      <th>Resulting <code>--test</code> value</th>
    </tr>
  </thead>

<tbody>

<tr>
<td>
Override default
</td>
<td>

```sh
node --test ./src/foo/*.test.js
```

</td>
<td>

`./src/foo/*.test.js`

</td>
</tr>

<tr>
<td>
Override default & Set additional path
</td>
<td>

```sh
node
  --test ./src/foo/*.test.js
  --test-path ./src/bar/*.test.js
```

</td>
<td>

`./src/foo/*.test.js`
`./src/bar/*.test.js`

</td>
</tr>

<tr>
<td>
Defaults + Additional paths
</td>
<td>

```sh
node
  --test
  --test-path ./src/foo/*.test.js
  --test-path ./src/bar/*.test.js
```

</td>
<td>

`**/*.test.{cjs,cts,mjs,mts,js,ts}`, [etc](https://nodejs.org/api/test.html#running-tests-from-the-command-line)
~`./src/foo/*.test.js`~
~`./src/bar/*.test.js`~

which reduces to only the default

</td>
</tr>

<tr>
<td>
No defaults & Additional paths
</td>
<td>

```sh
node
  --test-path ./src/foo/*.test.js
  --test-path ./src/bar/*.test.js
```

</td>
<td>

`./src/foo/*.test.js`
`./src/bar/*.test.js`

</td>
</tr>

</tbody>
</table>
