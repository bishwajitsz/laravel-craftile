# Release Notes

## v0.1.0

First release of `tradifylabs/laravel-craftile`, a maintained fork of
`craftile/laravel` republished under a TradifyLabs vendor name so it can be
depended on from other projects.

Based on upstream `craftile/laravel` v0.9.2 (MIT). The `src/`, `config/`, and
`resources/` trees are unchanged from that release. Only `composer.json`
differs:

- `illuminate/support`: `^11.0|^12.0` → `^13.0`
- `craftile/core`: `self.version` → `^0.9`

Upstream pins `craftile/core` to `self.version`, which forces core and laravel to
be released at identical version numbers and blocks depending on core `^0.9`
alongside a newer laravel integration. This fork decouples the two.

The PHP namespace (`Craftile\Laravel`), service provider, facades, config keys,
and publish tags are unchanged, so this is a drop-in replacement for
`craftile/laravel`. Do not install both at once — they provide the same
namespace.
