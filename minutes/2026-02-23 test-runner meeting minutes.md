# 23 February 2026

## Attendees

* Alex Vespa [@vespa7](https://github.com/vespa7)
* Aviv Keller [@avivkeller](https://github.com/avivkeller)
* Diogo de Souza [@sozua](https://github.com/sozua)
* Jacob Smith [@JakobJingleheimer](https://github.com/JakobJingleheimer) (chair)
* Jordan Harband [@ljharb](https://github.com/ljharb)
* Pietro Marchini [@pmarchini](https://github.com/pmarchini)

## Topics

* test_runner: run afterEach for runtime t.skip() [nodejs/node#61525](https://github.com/nodejs/node/pull/61525)
* doc(proposal): un/break --test [nodejs/test-runner#13](https://github.com/nodejs/test-runner/pull/13)
  * `--watch path` + `--test`
    * `node main.js --test test.js` runs `main.js` and passes `--test test.js` as custom args (NOT a node arg). **test mode is not engaged.**

## Outcomes

* Discussion on `nodejs/node#61525` is resolved: This exists not uncommonly in the wild, and the concern has been withdrawn.
* `nodejs/test-runner#13`: `--watch` is just a boolean when combined with `--test` (but in order to not encounter the caveated situation, `--test` must occur before `--watch` in the CLI arg list). Thus, if both are set in a config file (where there is no sequence), and `--watch`'s value is not boolean, that throws.

## Todos

* @JakobJingleheimer: Update previous meeting minutes to better state the `t.skip()` item raised. Once done, PR should be good to merge.
* @JakobJingleheimer: Poke René about [nodejs/node#60751](https://github.com/nodejs/node/pull/60751) (it seems his concern has been addressed and PR needs re-review).
* @JakobJingleheimer: Add caveat note to `nodejs/test-runner#13` about "passing" a value to `--watch` before triggering test mode.
* @JakobJingleheimer: Create poll for finding a new meeting time.
* Check `nodejs/node#61525` for outstanding concerns or unresolved discussions; if none, land the PR.