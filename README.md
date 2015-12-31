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

# GENERAL BEST PRACTICES

* Start by carefully reading the entire [npm shrinkwrap docs](https://docs.npmjs.com/cli/shrinkwrap)
* Never commit your `node_modules` directory (add it to `.gitignore`)
* Always commit your `npm-shrinkwrap.json` file.
* Always carefully review any changes to `npm-shrinkwrap.json` before
  committing it, and ensure that:
  1. Any diffs are expected
  1. Understand why they are there (the latter may be unachievable depending
  on your level of knowledge).
* (optional, but a good idea) Do not install any packages globally, even on
  development workstations.  Instead do one of the following:
  * On workstations, use [direnv](http://direnv.net/) on
    workstations to add `node_modules/.bin` to your path.  E.g., make a
    `.envrc` file in the root of your project with the contents
    `PATH_add node_modules/.bin`, and run follow instructions to set up
    direnv on your workstation.
  * Write an npm wrapper script and and use `npm run`
  * Write an executable script in node with a shebang
    for node (`#!/usr/bin/env node`) to the top of the script.  

# USE CASES:

## Creating initial shrinkwrap file

1. Make sure you have no existing `npm-shrinkwrap.json` file.
1. Make sure you have a clean working copy and no un-pushed commits, and
  `node_modules` in your `.gitignore`.
1. Remove (move and keep a copy for reference) your `node_modules` directory
  from your project.
1. Run `npm i` (NO global option!) to recreate `node_modules` containing ONLY the
  packages specified by your `package.json`
1. Run `npm shrinkwrap --dev` to create the `npm-shrinkwrap.json` shrinkwrap
  file.
1. If you get any errors, especially "extraneous" errors, see the "HANDLING
  ERRORS" section below, then start over.
1. Commit ***BUT DON'T PUSH YET*** that new shrinkwrap file.
1. Again, remove (or backup) your `node_modules` directory, and do another
  `npm i`.  This time, it will use your shrinkwrap file to install, instead
  of the package.json.
1. Check out the diffs of `npm-shrinkwrap.json`.  You'll probably several
  differences in the `resolved` entries.
1. Go ahead and commit (or amend the previous commit, if you want a single
  commit) this updated version of `npm-shrinkwrap.json`.
1. One final (third) time, delete `node_modules` and do another `npm i`.
  This time, you should have NO changes.

## Upgrading or installing a package dependency (conservatively)

1. Make sure you have a clean working copy and no un-pushed commits, and
  `node_modules` in your `.gitignore`.
1. Remove (move and keep a copy for reference) your `node_modules` directory
  from your project.
1. Run `npm i` (NO global option!) to recreate `node_modules` containing ONLY the
  packages specified by your `npm-shrinkwrap.json`
1. DELETE your `npm-shrinkwrap.json` but DO NOT commit the deletion (you'll)
  recreate it in a minute)
1. Make the necessary changes to your `package json` to specify the
   new/updated dependency.  Only change a single dependency at a time, ideally.
1. Run `npm i` to install the new/updated dependency in your `node_modules`
1. Run `npm shrinkwrap --dev`.
1. Do a diff of the changes to `npm-shrinkwrap.json` compared to the
  currently-committed version.  The ONLY changes should be expected
  changes related to your new/updated package.
1. If they look legit, commit the changees to `npm-shrinkwrap.json`
  ***BUT DON'T PUSH YET***
1. Again, remove (or backup) your `node_modules` directory, and do another
  `npm i`.
1. Check out the diffs of `npm-shrinkwrap.json`.  You may see
  differences in the `resolved` entries.
1. Go ahead and commit (or amend the previous commit, if you want a single
  commit) this updated version of `npm-shrinkwrap.json`.
1. Now you should be safe to push your changes, and be confident that
  future invocations of `npm shrinkwrap --dev` won't introduce any
  spurious changes to the `resolved` entries.

## Making an existing shrinkwrap environment have pristine packages

1. This is identical to the process above for creating the initial shrinkwrap
  file, just skip the first step saying to make sure you have no existing
  shrinkwrap file.

## Installing fresh from shrinkwrap on a new environment

1. Check out the project.
1. Delete any existing `node_modules` directory (it should already
  be `.gitignore`'d), so should have no changes to commit).
1. Run `npm i`, which will install everything as specified by the
  current `npm-shrinkwrap.json` (ignoring anything in `package.json`)

# HANDLING ERRORS

I'm not positive these are the right things to do.  Feedback is welcome.

## `npm ERR! extraneous: ...`

* This occurs when you have an "extraneous" dependency not referenced by
  `package.json`, as described in the [shrinkwrap docs]().  This
  can occur because you manually installed it, or because of some post install hooks
  [as described in this issue](https://github.com/paulmillr/chokidar/issues/92)
* The easiest (but perhaps not correct) solution is to simply delete the
  offending entry out of your node_modules directory with
  `rm -rf node_modules/packagename`
* If you didn't manually install the dependency, then you should track down
  what caused it to be installed (possibly a post-install hook, possibly
  in one of your dependencies) and stop that from happening.  

## Network errors

* These may look like this:
```
  npm ERR! 0-byte tarball
  npm ERR! Please run `npm cache clean`
```
* Or something about network errors
* In this case, check your network (`ping google.com`), then start whatever
 you were doing from scratch and try again.
