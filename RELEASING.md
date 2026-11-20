# Releasing a plugin

Plugins release themselves. Every push to a plugin repo's `main` runs its **Release** workflow, which reads the [Conventional Commits](https://www.conventionalcommits.org) since the last `v*` tag:

| Commits since the last tag | Release |
|---|---|
| a breaking change: `feat!:`, `fix(api)!:`, or a `BREAKING CHANGE:` footer | next major; next minor while the version is `0.x` |
| `feat:` | next minor |
| `fix:`, `perf:` or `revert:` | next patch |
| only `chore`, `docs`, `ci`, `test`, `refactor`, `build` or `style` | none |

Commit messages in these repos must be Conventional Commits; Release warns about any other message and does not count it. A commit that sets `manifest.json`'s version to an untagged value releases exactly that version (when it also carries a releasing commit), which is how to jump to `1.0.0`.

Everything runs as the `attmonk-ci` GitHub App:

1. **Release** (`release-plugin.yml` in this repo) plans the version and notes, bumps `manifest.json`, runs the plugin's checks, commits `chore(release): <name> <version>`, makes an annotated tag `v<version>` holding the notes, and pushes both. The `chore(release)` commit releases nothing, so the push does not loop.
2. The tag push starts the plugin's tag build, which builds the three binaries and publishes the GitHub release with `checksums.txt`.
3. The tag build's last job (`publish-plugin.yml` in this repo) checks the binaries against `checksums.txt`, copies the tag's notes into the release, updates this catalog with `scripts/update-entry.mjs` (which runs the validator), and pushes the catalog.

To release by hand (for example a maintenance build with no releasing commits), open the plugin repo's **Actions** tab, choose **Release**, then **Run workflow** on `main`. Leave **version** blank to take the version from the commits (at least a patch), or type one.

| Plugin repo | Release commit / tag title | Notes | Catalog asset prefix | Old checksums |
|---|---|---|---|---|
| `silo-plugin-dispatcharr` | `Dispatcharr <version>` | tag and release | `dispatcharr-<version>-` | replaced; entry synced from the plugin manifest |
| `silo-plugin-xtream-library` | `XC for Silo <version>` | `docs/releases/v<version>.md`, written from the commits unless already on `main` | `xtream-<version>-` | kept |
| `silo-plugin-app-links` | `App Links <version>` | tag and release | `app-links-<version>-` | replaced |
| `silo-plugin-local-artwork` | `Local Artwork <version>` | tag and release | `plugin-` | replaced |
| `silo-plugin-watchprovider-trakt` | `Trakt <version>` | tag and release | `trakt-<version>-` | replaced |

A delayed older tag build cannot downgrade a newer catalog entry: the publisher compares release versions before changing the manifest or checksums.

Check the run's summary for the planned version, the notes and the catalog commit. The "Validate Catalog" workflow reruns the validator on the push. A release whose push loses a race with a newer commit fails without pushing anything; the run for the newer commit, queued behind it, releases both.

## Setup the workflows need

- The organization secret `ATTMONK_CI_APPKEY` holds a private key of the CI GitHub App (app ID 5324952) and must allow all repositories, including recreated repository IDs. Workflows also accept `ATTMONK_CI_APP_KEY` for compatibility. The App is `attmonk-ci`, with the original installation and private key.
- Actions must be enabled in each plugin repo. A new fork runs no workflows until someone enables them on its Actions tab.

## Adding a plugin to the catalog

Add the plugin to `catalogDisplayNotes` in `scripts/validate-catalog.mjs` first; the validator rejects unlisted plugins. Its first catalog publish then creates the entry from the plugin's `manifest.json` (without its presentation text). Give the plugin's tag build a final job like the others:

```yaml
  catalog:
    needs: release
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v')
    uses: attmonk/silo-plugins/.github/workflows/publish-plugin.yml@main
    with:
      plugin-id: silo.attmonk.<name>
      display-name: <Name>
      asset-prefix: <name>-{version}-
    secrets: inherit
```

Release assets must carry a plugin-specific prefix: `checksums.txt` here is keyed by file name.

## By hand

When a workflow cannot run, release locally instead, as attmonk-ci and through the app's credentials:

1. Bump `manifest.json` on `main`, commit as `chore(release): <name> <version>`, make an annotated tag `v<version>` with the notes, and push `main` and the tag together.
2. Download `checksums.txt` and the three binaries from the release and check them with `shasum -a 256 -c`.
3. Update this catalog:

```bash
node scripts/update-entry.mjs --plugin-id <catalog plugin id> --repo <plugin repo name, without attmonk/> \
  --version <version> --asset-prefix <prefix> --checksums <downloaded checksums.txt> \
  [--replace-old] [--plugin-manifest <plugin>/manifest.json [--sync-manifest]]
```

The script changes only the version, checksums and download URLs unless `--sync-manifest` is set, then runs the validator. The catalog's display names and descriptions are its own, pinned in `scripts/validate-catalog.mjs`; they intentionally differ from the plugins' manifests, so leave them alone unless a rename is wanted, and change both files together.

Commit as `chore(catalog): update <plugin name> to <version>` (several plugins: one commit naming each) and push `main`.

## Catalog contributor attribution

Catalog commits use the verified App address
`342793482+attmonk-ci[bot]@users.noreply.github.com` with author name
`attmonk-ci`. The publisher fixes this identity so an organization email or
repository variable cannot attribute catalog commits to a person. Local catalog
checkouts set `attmonk.requireBotAttribution=true`; the commit guard enforces the
same identity. Published version tags remain unchanged when default-branch
history is replaced, preserving existing Go module checksums.
