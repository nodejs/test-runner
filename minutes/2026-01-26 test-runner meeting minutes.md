# 26 January 2026

## Attendees

* Alex Vespa [@vespa7](https://github.com/vespa7)
* Aviv Keller [@avivkeller](https://github.com/avivkeller)
* Chemi Atlow [@atlowChemi](https://github.com/atlowChemi)
* Ethan Arrowood [@Ethan-Arrowood](https://github.com/Ethan-Arrowood)
* Jacob Smith [@JakobJingleheimer](https://github.com/JakobJingleheimer) (chair)
* Pietro Marchini [@pmarchini](https://github.com/pmarchini)

## Topics

* [Test suite timeout](https://github.com/nodejs/node/issues/59701): Wether test-cases should inherit the suite's timeout or use the global default
* Pietro proposed an interactive test mode (similar to Jest's).

## Outcomes

* Test suite timeout
  * Jacob proposed that when a suite has a timeout, children (that do not explicitly set their own timeout) have no timeout. They run until the suite hits its timeout, at which point it cancels all the children.
* Interactive test mode
  * Ethan & Jacob were skeptical about the usefulness of adding UI for this (but like the core concept): it's already easily possible with smarter globs and potentially additional paths. Those features are perhaps less well-known, and Jacob pointed out the bizarre behaviour of `--test` (to Ethan's surprise). They felt the time would better spent improving and "fixing" the current tools and their interop.

# Todos

* Jacob to open a proposal for un/breaking `--test` and its potential interop with `--watch`.
