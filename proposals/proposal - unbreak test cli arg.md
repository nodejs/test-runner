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

```sh
node
  --watch
  --watch-path ./src/**/*.ts
  --watch-path ./test/**/*.ts
```

## Proposed options

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
```sh title='package.json "main" + an additional path'
node --watch ./src/**/*.js
```
```sh title='explicit entrypoint + additional paths'
node
  --watch ./assets/**
  --watch ./src/**/*.js
  ./src/not-pjson-main.js
```

An explicit entrypoint, like all node commands, must come last and must be a relative or absolute path (not a glob/pattern).

`test` would then work the same way:

```sh title='reads "main" etc from package.json'
node --test
```
```sh title='package.json "main" + an additional path'
node --test ./src/**/*.test.js
```
```sh title='explicit entrypoint + an additional path'
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  ./src/not-pjson-main.js
```

#### All together

`--watch` receives a clone of `--test`'s resolved value(s) **plus** `--watch`'s own values—with 1 exception: when both `test` and `watch` modes are enabled, `--watch` does not include a default entrypoint (it's irrelevant and it would likely result in perf waste at best, and unexpected behaviour at worst).

All examples are sequence-independent (all within the same heading behave the same).

##### Watch + test paths & main entrypoint:

```sh
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  --watch
  ./src/not-pjson-main.js
```
```sh
node
  --watch
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  ./src/not-pjson-main.js
```

##### Without an entrypoint (just paths):

```sh
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  --watch
```
```sh
node
  --watch
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
```

##### _Additional_ watch paths

```sh
node
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
  --watch ./src/**/*.js
```
```sh
node
  --watch ./src/**/*.js
  --test ./src/foo/*.test.js
  --test ./src/bar/*.test.js
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

`./src/**/*.test.(c|m)?(j|t)s`
~`./src/foo/*.test.js`~
~`./src/bar/*.test.js`~

which reduces to only the default: `./src/**/*.test.(c|m)?(j|t)s`

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
