# pw-module-page-hit-counter

Castus fork of [flipzoom/PageHitCounter](https://github.com/flipzoom/PageHitCounter), a ProcessWire module that counts page views.

> This README lives in `.github/` so that upstream's own [`README.md`](../README.md) stays untouched and never conflicts when we merge upstream changes.

## What this is

- **Used by:** [castusdesign/intertrain](https://github.com/castusdesign/intertrain). It stores view counts in the `phits` field on resource, course-type and course-category pages. The Algolia indexer reads those counts to rank results by popularity.
- **Composer package:** `castusdesign/pw-module-page-hit-counter` (type `processwire-module`)
- **Installs to:** `public_html/site/modules/PageHitCounter/` (set by `extra.installer-name`)

## Branches

| Branch | Contains | Rule |
|---|---|---|
| `source` | Upstream history, exactly as published | **Never commit our changes here.** It only ever fast-forwards to an upstream tag. |
| `main` (default) | `source` + `composer.json`, `.gitattributes`, this README, and our patches | Every change is its own commit, listed below |

## Current base

- **Upstream:** https://github.com/flipzoom/PageHitCounter
- **Version:** `v2.0.1` (module version 201), commit `71f0508`
- **Our release tag:** `2.0.1-patch1`

## Our patches

| Commit on `main` | What it changes | Why |
|---|---|---|
| "Stop logging the tracking response to the browser console" | `PageHitCounter.js`: `console.info('Page Hit Counter: Tracked. ' + xhr.responseText)` becomes `console.info('Page Hit Counter: Tracked.')` | The tracker logged the endpoint's response body on every page view. Originally applied in intertrain `6ca3448`. |

**Only affects debug mode.** The module serves `PageHitCounter.min.js` unless `$config->debug` is on (`buildTrackingCode()` in `PageHitCounter.module`), and the minified file has no `console` calls. So in production this patch changes nothing.

**Can it be dropped?** Yes, once upstream stops logging `xhr.responseText` in `PageHitCounter.js`. Check with:

```sh
git fetch upstream --tags
git show <new-tag>:PageHitCounter.js | grep -n responseText
```

If that finds nothing, `git revert` the patch commit on `main` during the update.

## How to update

1. **One-time setup** in your clone:

   ```sh
   git remote add upstream https://github.com/flipzoom/PageHitCounter.git
   ```

2. **Fetch upstream and move `source` to the new release:**

   ```sh
   git fetch upstream --tags
   git switch source
   git merge --ff-only v2.0.2        # the new upstream tag
   git push origin source
   ```

   If `--ff-only` refuses, upstream rewrote history. Stop and look before forcing anything.

3. **Merge into `main`:**

   ```sh
   git switch main
   git merge source
   ```

   Resolve any conflict in `PageHitCounter.js` by keeping upstream's change and re-applying the patch above. Check whether the patch is still needed at all.

4. **Tag and push.** Use the upstream version plus a `-patchN` suffix while we carry patches. Composer treats `2.0.2-patch1` as a stable release newer than `2.0.2`.

   ```sh
   git tag 2.0.2-patch1
   git push origin main --tags
   ```

   If a new patch goes on without an upstream change, bump only the suffix, e.g. `2.0.1-patch2`.

## Rolling it out in Intertrain

In the Intertrain repo:

1. Run `docker compose exec app composer update castusdesign/pw-module-page-hit-counter`.
2. Commit the changed `composer.lock`.
3. In the admin, go to Modules > Refresh. It should report the new version and nothing missing.
4. Smoke-test:
   - A resource page loads and records a hit: the `phits` value goes up.
   - The admin page tree still shows hit counts.
5. Open a PR. Deploying is done by the team as usual.

## Licence

MIT, as upstream (see [`LICENSE`](../LICENSE)). Our patches are released under the same licence.
