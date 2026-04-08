# 08 April 2026

## Attendees

* Alex Vespa [@vespa7](https://github.com/vespa7)
* Aviv Keller [@avivkeller](https://github.com/avivkeller)
* Ethan Arrowood [@Ethan-Arrowood](https://github.com/Ethan-Arrowood)
* Jacob Smith [@JakobJingleheimer](https://github.com/JakobJingleheimer) (chair)

## Topics

* Proposal: adjust snapshot file label to account for subtest label (~breaking) [nodejs/test-runner#27](https://github.com/nodejs/test-runner/pull/27)
* Proposal: change default snapshot filename (~breaking) [nodejs/test-runner#26](https://github.com/nodejs/test-runner/pull/26)
* doc(proposal): suite-level hooks [nodejs/test-runner#25](https://github.com/nodejs/test-runner/pull/25)
* High-impact suggestions from Colin [nodejs/test-runner#21](https://github.com/nodejs/test-runner/issues/21)
* doc(proposal): un/break --test [nodejs/test-runner#13](https://github.com/nodejs/test-runner/pull/13)

## Outcomes

* `nodejs/test-runner#13` Proceeding with the broadly accepted current proposal and can potentially pursue the subcommand in future.
* `nodejs/test-runner#27` Let's do it
* `nodejs/test-runner#26` Let's do it
* `nodejs/test-runner#25` Let's do it

## Todos

* @JakobJingleheimer: to pick up & finish [nodejs/node#61598](https://github.com/nodejs/node/pull/61598) (toward code coverage stability à la `nodejs/test-runner#21`)
* @JakobJingleheimer: flesh out breaking mitigation for `nodejs/test-runner#27`
* @avivkeller: to write up proposal on alterning format of snapshot file (CJS vs ESM).
* @vespa7: to tackle implementation for `nodejs/test-runner#25`
