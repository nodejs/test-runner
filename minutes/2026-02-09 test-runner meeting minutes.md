# 09 February 2026

## Attendees

* Alex Vespa [@vespa7](https://github.com/vespa7)
* Ethan Arrowood [@Ethan-Arrowood](https://github.com/Ethan-Arrowood)
* Jacob Smith [@JakobJingleheimer](https://github.com/JakobJingleheimer) (chair)
* Jordan Harband [@ljharb](https://github.com/ljharb)
* Pietro Marchini [@pmarchini](https://github.com/pmarchini)

## Topics

* `t.skip()` Others (tape, etc) to not behave like this.
* standardising test runner design
  * https://github.com/WinterTC55/proposal-minimum-common-api/issues/32
  * https://github.com/WinterTC55/proposal-minimum-common-api/issues/68
* [doc(proposal): `expectFailure` label and/or matcher](https://github.com/nodejs/test-runner/pull/10)
* [doc(proposal): un/break `--test`](https://github.com/nodejs/test-runner/pull/13)

## Outcomes

* General agreement that `t.skip()`'s behaviour is wrong (likely an accident/bug/mistake, which the docs seem to support).
* General agreement that it would be good to standardise node's test-runner at the WinterTC55 level, if we can, but might be a very uphill battle. Perhaps better to audit our and others' design and adjust (and then take that to WinterTC).
* https://github.com/nodejs/test-runner/pull/10 ready to approve
* Un/break `--test`
  * option 3
    * leave `--watch` as-is
      * in `test` mode, its value (`--watch` only enables watch mode)
    * support `--watch-path` in test mode
    * only `--test` N-times 1-to-1 with arg list, making it position-independent

## Todos

* Decide if fixing `t.skip()` is a breaking change.
* Jordan to find & share his audit of test runner features to use as base-point.
* Jacob to incorporate "option 3" to https://github.com/nodejs/test-runner/pull/13 and tag team for review.
