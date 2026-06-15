# mago-mediawiki-style

![Files mago would change](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2Fchaotic-ground%2Fmago-mediawiki-style%2Fmain%2F.github%2Fbadges%2Fparity.json)

An experiment: maintain a [mago] configuration that reproduces the
[MediaWiki coding style][mw-style] (as enforced by
[mediawiki-codesniffer]), measured by running `mago format` against the
latest [MediaWiki core][mediawiki] source.

The goal is a `mago.toml` for which mago makes **no changes** to MediaWiki
core. CI runs weekly against the latest core and the latest mago, and
records how far away we are.

## Current status

Not reachable yet. Known differences that mago cannot currently be
configured away:

- MediaWiki puts spaces inside control-structure and call parentheses:
  `if ( $x )`, `foo( $arg )`. mago only offers
  `space-within-grouping-parenthesis`; there is no option for control
  structures, call/argument lists, or parameter lists.
- MediaWiki puts spaces inside array brackets: `[ 'key' => $value ]`.
- mago rewrites `#` line comments to `//`. mediawiki-codesniffer agrees
  with that direction, but core still contains `#` comments, so the
  rewrite inflates the diff.
- mago reflows broken method chains onto one call per line, while core
  often groups several calls on a line (e.g. `->caller( __METHOD__ )
  ->fetchRow()`), even with the `preserve-breaking-*` settings.
- mago rewraps multi-line conditions that fit within the print width and
  may add clarifying parentheses around arithmetic inside comparisons.
- `trailing-comma` is a single global boolean: mago either adds trailing
  commas to every multi-line structure or removes them from all of them,
  with no preserve option and no per-construct control. MediaWiki core
  commonly uses them in multi-line arrays but not in call argument lists,
  so either setting rewrites one side wholesale.

## Current baseline

The numbers below are written by the Parity workflow after every run.

<!-- METRICS:START -->

- Measured: 2026-06-15
- mago 1.30.0
- MediaWiki: [wikimedia/mediawiki@dc41709](https://github.com/wikimedia/mediawiki/commit/dc41709b76fff57e410df25c93b510b5b875b9f8)
- PHP version: 8.5
- Files mago would change: **5367** of 5557 PHP files
- 5367 files changed, 340453 insertions(+), 319866 deletions(-)

First 20 lines of the diff:

````diff
diff --git a/.phan/config.php b/.phan/config.php
index e86bb61..756db1d 100644
--- a/.phan/config.php
+++ b/.phan/config.php
@@ -7,7 +7,7 @@
 $cfg = require __DIR__ . '/../vendor/mediawiki/mediawiki-phan-config/src/config.php';

 // Unset the default value to make sure we use that from composer.json.
-unset( $cfg['minimum_target_php_version'] );
+unset($cfg['minimum_target_php_version']);

 $cfg['file_list'] = array_merge(
 	$cfg['file_list'],
@@ -18,7 +18,7 @@ $cfg['file_list'] = array_merge(
 		// @todo This isn't working yet, see globals_type_map below
 		// 'includes/Setup.php',
 		'tests/phpunit/MediaWikiIntegrationTestCase.php',
-		'tests/phpunit/includes/TestUser.php',
+		'tests/phpunit/includes/TestUser.php'
 	]
````

<!-- METRICS:END -->

## How it works

- `mago.toml` holds the candidate configuration, tuned to match the
  MediaWiki conventions wherever mago has a knob (tabs, same-line braces,
  single quotes, no space after casts, no trailing commas, and so on).
- The [Parity workflow](.github/workflows/parity.yaml) shallow-clones the
  latest `wikimedia/mediawiki` (cached between runs), installs the latest
  mago release, runs `mago format`, and writes the diff size to the job
  summary, the badge above, and the baseline section of this README.
  It never fails on a non-zero diff; it is a tracker, not a gate.

## Running locally

```sh
git clone --depth=1 https://github.com/wikimedia/mediawiki.git mediawiki
mago format --dry-run   # or: mago format && git -C mediawiki diff --shortstat
```

[mago]: https://github.com/carthage-software/mago
[mediawiki]: https://github.com/wikimedia/mediawiki
[mediawiki-codesniffer]: https://github.com/wikimedia/mediawiki-tools-codesniffer
[mw-style]: https://www.mediawiki.org/wiki/Manual:Coding_conventions/PHP
