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
* Managing differences between dev vs. prod environments/deploys, while still
 being able to use shrinkwrap to lock down dependencies in both cases.

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
* Try to only upgrade the minimal set of dependencies at a time (ideally
  only one), and do them each in a separate commit, including only
  the necessary changes to `package.json` and `npm-shrinkwrap.json`.
  If you want to subsequently squash this into the commit that actually
  uses this dependency, you can, but it's good to keep it isolated at first
  so you can easily see if it breaks anything in isolation and revert it.
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

Note: Running `npm prune` may be a much faster alternative
to deleting/moving the `node_modules` directory
in the steps below, but I'm not positive it can
be trusted to have the identical result,
and not include some extra entries in `npm-shrinkwrap.json`
that wouldn't be there if you would have
deleted the directory.  So, the safer but slower
method is shown.  Feedback is welcome.

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
1. ***UPDATE:***  Recent versions of NPM (>= 3.7 ???) seem to have resolved
   the "flapping `resolved` entries" problem, so if you never see any changes
   from the below steps, you can safely skip them and push now.
1. Again, remove (or backup) your `node_modules` directory, and do another
  `npm i`.  This time, it will use your shrinkwrap file to install, instead
  of the package.json.
1. Then do another `npm shrinkwrap --dev`
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
1. ***UPDATE:***  Recent versions of NPM (>= 3.7 ???) seem to have resolved
   the "flapping `resolved` entries" problem, so if you never see any changes
   from the below steps, you can safely skip them and push now.
1. Again, remove (or backup) your `node_modules` directory, and do another
  `npm i`.
1. Check out the diffs of `npm-shrinkwrap.json`.  You may see
  differences in the `resolved` entries.
1. Go ahead and commit (or amend the previous commit, if you want a single
  commit) this updated version of `npm-shrinkwrap.json`.
1. Now you should be safe to push your changes, and be confident that
  future invocations of `npm shrinkwrap --dev` won't introduce any
  spurious changes to the `resolved` entries.

## Making a new or existing shrinkwrap environment have pristine packages

1. Check out the project.
1. Delete any existing `node_modules` directory (it should already
  be `.gitignore`'d), so should have no changes to commit).
1. Run `npm i`, which will install everything as specified by the
  current `npm-shrinkwrap.json` (ignoring anything in `package.json`)

# HANDLING ERRORS

I'm not positive these are the right things to do.  Feedback is welcome.

## `npm ERR! extraneous: ...`

* This occurs when you have an "extraneous" dependency not referenced by
  `package.json`, as described in the
  [shrinkwrap docs](https://docs.npmjs.com/cli/shrinkwrap).  This
  can occur because you manually installed it, or because of some post install hooks
  [as described in this issue](https://github.com/paulmillr/chokidar/issues/92)
* First, determine if it's a postinstall problem or a manual install problem.
  If you were carefully following processes above regarding deletion of the 
  `node_modules` directory at appropriate points, then there should be no
  chance of having manually installed something.  So, optionally run through
  the process again carefully, verify you get the same error, then continue
  on and assume that it's due to a postinstall issue...
* The root of the problem is that the `npm shrinkwrap` command does not
  (as of npm <= 3.9) seem to be "aware" of packages that were installed
  in `node_modules` as a result of a postinstall hook of some dependency.
  Therefore it will fail with the "extraneous" error whenever it encounters
  one of these packages it doesn't expect.
* So, the only workaround I know is to explicitly add the "extraneous"
  dependenc(ies) as a top-level entry in your `package.json`.  You can pick
  whatever version constraint is appropriate, and add it to main or dev
  dependencies as appropriate.  Yes, this is lying and claiming your
  app has a direct dependency when it actually doesn't, and you'll have to 
  manually update/delete it if necessary in the future, but until
  the shrinkwrap command is fixed to handle this situation I don't know
  of any other workaround.  Please let me know if you have more insight :)

## Network errors

* These may look like this:
```
  npm ERR! 0-byte tarball
  npm ERR! Please run `npm cache clean`
```
* Or something about network errors
* In this case, check your network (`ping google.com`), then start whatever
 you were doing from scratch and try again.

# OTHER RELATED TOOLS

* Shrinkpack:
  * [shrinkpack](https://github.com/JamieMason/shrinkpack) lets you check in
    a copy of all your dependencies.
  * There's tradeoffs with this approach.  It really blows up your git repo
    size to check in all third party tools, that's why I stopped using bundle
    package on Rails projects.  Plus, I get the same build speedups by having
    my CI system cache my node_modules directory.  And, I still lock down all
    dependencies via shrinkwrap so I'm not exposed to floating version regressions
    like he says (or at least they are clear where they came from because I
    commit my shrinkwrap.json).  But on the third hand, it is nice to have a
    copy of all your dependencies in case
    [someone decides to yank them off the central repository](https://medium.com/@azerbike/i-ve-just-liberated-my-modules-9045c06be67c#.b9emltwfr).
    Not that that could ever happen.
    [Or break thousands of projects](http://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/)...

# UNANSWERED QUESTIONS

## What about avoiding install of dev-only dependencies in production?

AFAIK, this workflow will result in all dev dependencies being installed in
production, because when `npm i` installs based off an existing
`npm-shrinkwrap.json`, it doesn't know or distinguish which ones originally
came from the devDependencies section in `package.json`.
[this issue](https://github.com/npm/npm/issues/10863) is related to the
problem.  For now, I'm assuming that it's safe (although unnecessary and slow)
to just install everything in production, since npm's architecture specifies
all sub-dependencies in the dependency tree independently.  For reference,
see [RubyGems'](http://guides.rubygems.org/) approach, which forces a
single version of a given package to be used, and
[Elm's package manager](http://elm-lang.org/blog/announce/package-manager),
which attempts to maximize sharing.
