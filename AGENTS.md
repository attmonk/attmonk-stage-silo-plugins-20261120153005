# ATTMonk catalog and plugin helpers

This repository publishes the ATTMonk Silo plugin catalog, shared Go helpers,
and reusable release workflows.

## Commit identity

Catalog commits must link to the verified GitHub App bot:
`attmonk-ci <342793482+attmonk-ci[bot]@users.noreply.github.com>`.
Configure this checkout with `git config attmonk.requireBotAttribution true`
and that exact `user.name`/`user.email`. Keep the commit guard enabled.
Use Conventional Commits and no co-author trailers.

Use the `attmonk-ci` GitHub App for pushes, tags, releases and workflow runs.
The App ID is 5324952; workflow credentials come from `ATTMONK_CI_APP_KEY`.
Do not override catalog author email with a personal address or repository variable.

## Publication and verification

Before pushing, fetch and compare with origin to preserve concurrent updates.
Run `go test ./...`, `go vet ./...`, `node scripts/validate-catalog.mjs`,
and the relevant script tests. Validate workflow edits with actionlint.

Keep published Go module tags and release assets immutable. The default branch
has a fresh ATTMonk root; archived version tags retain their existing snapshots
so plugin dependency checksums remain valid.

Keep catalog URLs, versions, IDs and checksums paired with verified release
assets. Read RELEASING.md before changing release or catalog automation.

## ATTMonk bot attribution and clean history

Use `attmonk-ci <342793482+attmonk-ci[bot]@users.noreply.github.com>` for both
author and committer. Set `attmonk.requireBotAttribution=true` in this checkout
and keep the commit guard enabled. Use Conventional Commits and no co-author
trailers. Pushes, tags, workflow runs and releases use the existing ATTMonk
GitHub App; CI credentials accept `ATTMONK_CI_APPKEY` or `ATTMONK_CI_APP_KEY`.

Keep history limited to clean ATTMonk snapshots and subsequent reviewed work.
Do not merge or push retired histories, branches or tags. Import upstream work
through reviewed patches or source snapshots so retired ancestry is not restored.
Legacy deployment compatibility values belong in local/server environment
configuration, never tracked source. Preserve runtime identities and storage
together when migrating existing installations.
