# 12 January 2026

## Attendees

* Alex Vespa [@vespa7](https://github.com/vespa7)
* Jacob Smith [@JakobJingleheimer](https://github.com/JakobJingleheimer) (chair)
* Jordan Harband [@ljharb](https://github.com/ljharb)
* Moshe Atlow [@MoLow](https://github.com/MoLow)
* Pietro Marchini [@pmarchini](https://github.com/pmarchini)
* Vas Sudanagunta [@vassudanagunta](https://github.com/vassudanagunta)

## Topics

* New team repo https://github.com/nodejs/test-runner
* Team Kanban: https://github.com/orgs/nodejs/projects/14
* Name for "expect failure": https://github.com/nodejs/test-runner/discussions/1
* Jacob proposed `flaky`, per https://github.com/nodejs/test-runner/discussions/1#discussioncomment-15468740

## Outcomes

* "Expect failure" feature:
  * Consensus reached for name: `expectFailure".
  * Widespread agreement on feature behaviour and use-cases.
    * Assertion inversion is perhaps not the targeted audience, but there no reason to get in its way. Documentation (likely a Learn article) may call out that use-case, but it's generally accepted to be not the focus.
* Consensus that `flaky` is a desired feature and general agreement on its behaviour/scope.

## Todos

* Need review (@nodejs/test-runner):
  * https://github.com/nodejs/node/pull/56490
  * https://github.com/nodejs/node/pull/59732
  * https://github.com/nodejs/node/pull/59700
* Write feature proposal (@JakobJingleheimer) for `flaky`
  * @JakobJingleheimer to support @vespa7 on implementation
