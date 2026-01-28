## Problem

We noticed that in the test runner documentation the `it()` and `describe()` aliases are introduced after they are already used in the *“Expecting tests to fail”* example. This makes the reading flow confusing for first-time readers.

This ordering issue is addressed by the following PR:
https://github.com/nodejs/node/pull/61567

While working on this, we also noticed a broader issue: there is no clear rationale for when examples use `test()` versus `it()` / `describe()`. As a result, the examples across the documentation are not fully consistent.

In particular, clarity would improve by keeping all flag-based options for `test()` together (`skip`, `todo`, `expectFailure`, `only`) and using a consistent style in those examples.

## Proposed solutions

### Option 1: Use `test()` for flag-based examples

Change the `expectFailure` example to use `test()` instead of `it()`.  
This keeps it consistent with the `skip` and `todo` examples and makes the documentation easier to learn.

### Option 2: Introduce `it()` / `describe()` earlier

If `it()` and `describe()` are considered common enough that readers should get used to them early, move the aliases section to the top of the document.  
After that, examples for other features can use either `test()` or `it()`.
