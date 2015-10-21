# Dependency License Auditing with SPDX

:boy: My open source project uses `${LICENSE}`. It's free!

:cop: Wait, what about all this other code in your dependencies?

:boy: Err, not sure...

:running:

:see_no_evil:
:hear_no_evil:
:speak_no_evil:

## NPM: Standing on the shoulders of dwarves

Let's whip up a basic web app stack...
```
npm init
npm install --save \
  express browserify \
  babel standard \
  karma mocha chai
npm ls | wc -l
# >1400 dependencies! whoa..
```

## Dependency Hell: Unfixable Bugs

> *Those who would give up Semantic Versioning for a little temporary safety deserve neither.* – Benjamin Franklin

- Dependency is not the latest version. Only `package@latest` is ever fixed. No one back-ports fixes to old versions. NPM default of `^` semver is best intention but road to hell.

  If I'm using `dep@^3.x.x` but `latest` is `v5.0.0` then a patch for `dep` will not be available to my app.

- Bug exists in a sub-dependency whose version range is capped. Sacrificing progress for safety.

  I'm using `dep@latest` which uses `subdep@1.0.0`. My fix for a bug in `subdep` will never go into `dep@latest`.

- Maintainers neglect packages. PRs are ignored. Versions are not published. Unreasonable friction to fork entire dependency tree.

**Solution?** Nope. Best/hippest practice today is to use [Greenkeeper](http://greenkeeper.io), a slightly more GitHub-integrated version of [Gemnasium](https://gemnasium.com), [David](https://david-dm.org), or [VersionEye](https://www.versioneye.com). Still creates a lot of [Shit Work](http://zachholman.com/posts/shit-work/). `#foodforthought`

## License Hell

NPM used to have a `licenses` property in `package.json`.
- Deprecated
- Widely abused and confused
- Inconsistent
- Typo `licences`, `lisences`, ...

Some repos follow an unspoken convention like `[LICENSE|COPYING][.md|.txt]`. Sometimes contains license text, sometimes a link, sometimes arbitrary codename.

In legal disputes ambiguity is not your friend.

NPM now only uses the `license` property. And it will complain if you don't use it.

## Are we legal yet?

Probably not, but it's getting better...

NPM has a reasonable default:

```
npm init
grep license package.json
  "license": "ISC",
```

What about MIT, BSD, CC, Apache, GPL, ...?

They are mostly compatible. KISS. Copyleft caveat emptor. Attribution PITA. CC-NC and CC-ND are evil.

TL;DR Just use ISC. :neckbeard::fire:

## Software Package Data Exchange (SPDX)

This problem was faced and solved long ago by Linux distros so they got together and solved it through the Linux Foundation.

Et voilà: [SPDX 2.0](http://spdx.org/sites/spdx/files/SPDX-2.0.pdf) (trigger warning: PDF)

### License Identifiers

Consistent short-codes for software licenses with canonical license text.

Examples: `ISC`, `MIT`, `CC-BY-4.0`, `Apache-2.0`, `GPL-3.0`

<http://spdx.org/licenses/>

No more...
- license file spam
- sneaky modifications
- outdated copyright timestamps
- evaporating URLs

### License Expressions

Formal syntax to dual-license code, offer license exceptions, etc.

Examples: `(ISC OR WTFPL)`, `(LGPL-2.1 AND CC-BY-2.5)`, `(GPL-2.0+ WITH Bison-exception-2.2)`

<https://www.npmjs.com/package/spdx>

## `node-license-validator`

```
npm install node-license-validator
node-license-validator --deep --allow-licenses ISC
```
615 invalid licenses! :scream:

Let's apply some common sense...
```json
{
  "deep": true,
  "production": true,
  "allow-licenses": [
    "ISC",
    "WTFPL",
    "Unlicense",
    "MIT",
    "BSD-2-Clause",
    "BSD-3-Clause",
    "Apache-2.0",
    "CC-BY-3.0"
  ]
}
```
55 invalid licenses. :astonished:

Pretty bad but getting better. Let's start solving the dependency hell.

Example: <https://github.com/cofounders/lgtm/issues/2>

![Screenshot of example GitHub issue to track dependency licensing chores](http://i.imgur.com/WWyEYfF.png)

This will take time but it is a one-time effort that benefits the community.

And in the mean time we can ~~cheat~~ exempt.
```json
{
  "allow-package": [
    "wordwrap@0.0.2",
    "jsonify@0.0.0",
    "indexof@0.0.1",
    "optimist@0.2.8||0.6.1"
  ]
}
```

### Protip: Automate your workflow

Add license audits to your CI to break the build, not the law.

`package.json`
```json
{
  "scripts": {
    "test": "mocha && standard && node-license-validator",
    "watch": "watch-spawn -p './package.json' node-license-validator"
  }
}
```

### Bonus: Help others

Dependency version bump pull request spam:

![Recent GitHub activity log for cbas](http://i.imgur.com/Ht30Fa1.png)

## Appendix

### Public Domain

SPDX does not have a license identifier for Public Domain. There is no clear solution for the `license` property and Public Domain content.

- [How to publish into the Public Domain](https://github.com/douglascrockford/JSON-js/pull/70#issuecomment-147403471)

- [SPDX statement on dealing with Public Domain](http://wiki.spdx.org/view/Legal_Team/Decisions/Dealing_with_Public_Domain_within_SPDX_Files)

### Unlicensed (All Rights Reserved)

From the NPM Documentation on the [package.json license property](https://docs.npmjs.com/files/package.json#license).

To make it explicit that a package is not free and open source, npm supports the following exception to SPDX:

```json
{ "license": "UNLICENSED" }
```

Consider also setting `"private": true` to prevent accidental publication.

### Custom licenses

If you are using a license that hasn't been assigned an SPDX identifier, or if you are using a custom license, use the following valid SPDX expression:

```json
{ "license" : "SEE LICENSE IN <filename>" }
```

Then include a file named `<filename>` at the top level of the package.

### Other Tools

[VersionEye](https://www.versioneye.com) is a dependency versioning SaaS with a license auditing feature that supports several platforms.

[Pivotal License Finder](https://github.com/pivotal/LicenseFinder) is a command line tool to detect dependency licenses and check them against a whitelist.
