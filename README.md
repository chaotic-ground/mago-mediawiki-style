# mago-mediawiki-style

An experiment: maintain a [mago] configuration that reproduces the
[MediaWiki coding style][mw-style] (as enforced by
[mediawiki-codesniffer]), measured by running `mago format` against the
latest [MediaWiki core][mediawiki] source.

The goal is a `mago.toml` for which mago makes **no changes** to MediaWiki
core. CI runs weekly against the latest core and the latest mago, and
records how far away we are.

## Current status

Not reachable yet. The dominant blocker is that MediaWiki puts spaces
inside parentheses and array brackets:

```php
if ( $x ) {
	foo( $arg );
	$a = [ 'key' => 'value' ];
}
```

mago only offers `space-within-grouping-parenthesis`; there are no
options for control structures, call/parameter lists, or array brackets.
Until upstream grows such options, nearly every file in core differs.

Baseline (mago 1.29.0, 2026-06): 5349 of 5540 PHP files changed,
roughly +339k/-319k lines.

## How it works

- `mago.toml` holds the candidate configuration, tuned to match the
  MediaWiki conventions wherever mago has a knob (tabs, same-line braces,
  single quotes, no space after casts, no trailing commas, and so on).
- The [Parity workflow](.github/workflows/parity.yaml) shallow-clones the
  latest `wikimedia/mediawiki` (cached between runs), installs the latest
  mago release, runs `mago format`, and reports the diff size in the job
  summary. It never fails on a non-zero diff; it is a tracker, not a gate.

## Running locally

```sh
git clone --depth=1 https://github.com/wikimedia/mediawiki.git mediawiki
mago format --dry-run   # or: mago format && git -C mediawiki diff --shortstat
```

[mago]: https://github.com/carthage-software/mago
[mediawiki]: https://github.com/wikimedia/mediawiki
[mediawiki-codesniffer]: https://github.com/wikimedia/mediawiki-tools-codesniffer
[mw-style]: https://www.mediawiki.org/wiki/Manual:Coding_conventions/PHP
