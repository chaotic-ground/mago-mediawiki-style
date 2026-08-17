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
- mago collapses a multi-line condition that fits within the print width
  onto a single line. `preserve-breaking-binary-expression` keeps the
  breaks, but indents the continuation lines differently from core, which
  changes more lines than the collapse does.
- `trailing-comma` is a single global boolean: mago either adds trailing
  commas to every multi-line structure or removes them from all of them,
  with no preserve option and no per-construct control. MediaWiki core
  commonly uses them in multi-line arrays but not in call argument lists,
  so either setting rewrites one side wholesale.

## Current baseline

The numbers below are written by the Parity workflow after every run.

<!-- METRICS:START -->

- Measured: 2026-08-17
- mago 1.46.0
- MediaWiki: [wikimedia/mediawiki@357281a](https://github.com/wikimedia/mediawiki/commit/357281a8caa63c3c38de445006e8d77ec62957bf)
- PHP version: 8.5
- Files mago would change: **5434** of 5626 PHP files
- 5434 files changed, 342902 insertions(+), 322954 deletions(-)

First 20 lines of the diff:

````diff
diff --git a/.phan/config.php b/.phan/config.php
index e86bb61..f36c0a5 100644
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

## Use in your project

`style.toml` holds only the formatter rules, so other projects can depend on
it. Add this repository as a Composer VCS source, require it, then `extends`
the installed file from your own `mago.toml`:

```json
"repositories": [
	{ "type": "vcs", "url": "https://github.com/chaotic-ground/mago-mediawiki-style" }
],
"require-dev": {
	"chaotic-ground/mago-mediawiki-style": "^0.1"
}
```

```toml
extends = "vendor/chaotic-ground/mago-mediawiki-style/style.toml"
php-version = "8.1"

[source]
paths = ["includes", "maintenance"]
```

mago reproduces only part of the MediaWiki style (see the known differences
above), so pair it with [mediawiki-codesniffer] for the sniffs mago cannot do.

## Running locally

```sh
git clone --depth=1 https://github.com/wikimedia/mediawiki.git mediawiki
mago format --dry-run   # or: mago format && git -C mediawiki diff --shortstat
```

[mago]: https://github.com/carthage-software/mago
[mediawiki]: https://github.com/wikimedia/mediawiki
[mediawiki-codesniffer]: https://github.com/wikimedia/mediawiki-tools-codesniffer
[mw-style]: https://www.mediawiki.org/wiki/Manual:Coding_conventions/PHP
