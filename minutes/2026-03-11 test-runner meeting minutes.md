# 11 March 2026

## Attendees

* Diogo de Souza [@sozua](https://github.com/sozua)
* Jacob Smith [@JakobJingleheimer](https://github.com/JakobJingleheimer) (chair)
* Pietro Marchini [@pmarchini](https://github.com/pmarchini)

## Topics

* doc(proposal): un/break --test [nodejs/test-runner#13](https://github.com/nodejs/test-runner/pull/13)
  * change test flag to subcommand? `node --test` → `node test`
* High-impact suggestions from Colin [nodejs/test-runner#21](https://github.com/nodejs/test-runner/issues/21)

## Outcomes

* From a technical standpoint, a `test` subcommand appears to be much easier than we expected.
  * It's however not obvious why this is not used more. We suspect it might be due to the breaking change where `node test` would have run `node ./test.js` (or `node ./test/index.js`)

## Todos

* @JakobJingleheimer: To ask around about lack of subcommands
  * Antoine FtW: it's indeed because of the breaking change suspected in the above discussion.
* @JakobJingleheimer: To keep an ear to the ground for the TC39 discussion about relaxing the requirement for dynamic import to always return the same.
* Consolidate `defaultExport` & `namedExports` [nodejs/node#61727](https://github.com/nodejs/node/pull/61727) needs some reviews and some opinions on whether old or new should "win" a conflict.
