# npm-shrinkwrap-helper

This repo is an attempt to define a process for navigating the
[self-acknowledged dependency hell](https://github.com/npm/npm/wiki/Roadmap-area-of-focus%3A-dependency-hell)
of [Node Package Manager](https://www.npmjs.com/)  version 3 when working
on NodeJS apps.

The goal is to have a process which supports completely locking down
all dependency versions, to guarantee there is no chance of regression
or failure due to different versions being installed or used
when the app is run in different development or deployment environments.
E.g. just like [Bundler](http://bundler.io/) for the Ruby ecosystem.

The [current documentation for shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap)
as of 2015 is inadequate (and I'm not
interested in helping improve it, at least until I try to define a
working process myself, which is the purpose of this repo).  It fails
to mention several headache-inducing problems, including but not limited
to:

* Why ["extraneous deps" errors](https://github.com/paulmillr/chokidar/issues/92)
  occur, and how to resolve them.
* How to resolve the problem of the value of "resolved" entries
  in npm-shrinkwrap.json flapping across commits (depending on whether
  they were installed from scracth or installed based on an existing
  shrinkwrap specification).

For now, this is just a scratchpad for the process I'm attempting to define.
I may make scripts around the processes later.

# USE CASES:

## Creating initial shrinkwrap file

* ...

## Upgrading a package (conservatively)

* ...

## Making an existing development environment have pristine packages

* ...

## Installing fresh on a new environment

* ...
