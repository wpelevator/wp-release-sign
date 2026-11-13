# WP Release Sign

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-WP%20Release%20Sign-blue?logo=github)](https://github.com/marketplace/actions/wp-release-sign) [![Latest release](https://img.shields.io/github/v/release/wpelevator/wp-release-sign?include_prereleases&sort=semver)](https://github.com/wpelevator/wp-release-sign/releases) [![npm](https://img.shields.io/npm/v/@wpelevator/wp-release?label=wp-release)](https://www.npmjs.com/package/@wpelevator/wp-release)

GitHub Action that signs release files with Ed25519 using [`@wpelevator/wp-release`](https://www.npmjs.com/package/@wpelevator/wp-release). It's named after the [`wp-release sign`](https://wpelevator.github.io/docs/js/wp-release/latest/signing.html) command that it runs.

It writes a detached `<file>.sig` next to each file: a base64 Ed25519 signature of the raw SHA-384 digest of the file, the format that WordPress core's `verify_file_signature()` checks. Sign both the ZIP and the checksums manifest: the ZIP signature covers installs and updates, and the manifest signature covers the contents after unpacking.

Set `format: wordpress,minisign,ssh` to also write signatures for `minisign -V` and `ssh-keygen -Y verify` with the `file` namespace. All formats use the same Ed25519 key. The default remains `wordpress`.

## Usage

Use `true` or `false` for boolean inputs; the action validates these values before running the command.

These actions require a runner with the Node.js 24 action runtime. Keep self-hosted runners updated before using them.

Pin `uses:` to the latest released version from the [releases page](https://github.com/wpelevator/wp-release-sign/releases). Upgrades are intentionally manual: floating major or minor tags are not published.

```yaml
jobs:
  release:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v7

      - id: zip
        uses: wpelevator/wp-release-zip@0.6.0
        with:
          checksums: true

      - id: sign
        uses: wpelevator/wp-release-sign@0.3.1
        with:
          files: |
            ${{ steps.zip.outputs.zip-path }}
            ${{ steps.zip.outputs.checksums-path }}
          signing-key: ${{ secrets.WP_RELEASE_SIGNING_KEY }}
          format: wordpress,minisign,ssh
```

Attach the ZIP, the manifest and all the files listed in `steps.sign.outputs.signatures` to the release. Publish the `public-key` output wherever the signatures are checked, such as the `signing_key` of [update-client](https://github.com/wpelevator/update-client), and check a signed manifest with `wp-release verify --public-key`.

Sign the manifest after writing it and before zipping or deploying when it's shipped inside the plugin, because the signature covers the exact bytes of the file. See [Manifest inside the plugin](https://wpelevator.github.io/docs/js/wp-release/latest/checksums.html#manifest-inside-the-plugin).

The action itself needs only `contents: read` for checkout; signing uses the supplied secret. Set this permission on jobs using the step examples in this README.

## Inputs

| Input | Default | Description |
|---|---|---|
| `files` | | Files to sign, one per line. Relative paths resolve from the repository root. Required. |
| `signing-key` | `WP_RELEASE_SIGNING_KEY` env var | Base64 32-byte Ed25519 seed or 64-byte libsodium secret key. Pass it from a secret; it's masked in the logs and never printed. |
| `format` | `wordpress` | Comma-separated signature formats: `wordpress` (`.sig`), `minisign` (`.minisig`), `ssh` (`.sshsig`). |
| `force` | `false` | Overwrite existing signature files. |

## Outputs

| Output | Description |
|---|---|
| `signatures` | Absolute paths of the written signature files, one per line. |
| `public-key` | Base64 public key of the signing key. |
| `formats` | JSON array of the signature formats written, such as `["wordpress","minisign","ssh"]`. |
| `public-keys` | JSON object of the public key in `wordpress`, `minisign` and `ssh` forms. |

For example, `${{ fromJSON( steps.sign.outputs.public-keys ).ssh }}` is the OpenSSH `ssh-ed25519` public key. The existing `public-key` output remains a base64 key accepted by WordPress and `wp-release verify`.

## Job summary

Each run adds a section to the job summary of the run with the signature files and the public key in each selected format, so it can be copied to where the signatures are checked. A failed run adds its error instead, and the run fails. The paths are relative to the workspace.

## Without the action

When you can't use a GitHub Action, such as on another CI system or in a local release script, add the same tooling as a development dependency and run the CLI command that the action runs, passing the key in the `WP_RELEASE_SIGNING_KEY` environment variable from your CI secrets:

```bash
npm install --save-dev @wpelevator/wp-release
npx wp-release sign my-plugin.1.0.0.zip my-plugin.1.0.0.checksums.json
```

The files are the arguments of [`wp-release sign`](https://wpelevator.github.io/docs/js/wp-release/latest/signing.html) and `format` and `force` are `--format` and `--force`. There's no flag for the signing key, since it would show in the process list. Add `--verbose` to print the public key, which the action returns as the `public-key` output, or `--json` for the paths of the signatures. The CLI needs Node.js 22.12 or later, and has no other dependencies on the runner. Run `npx wp-release sign --help` for all options.

## How it works

[`src/run.ts`](src/run.ts) reads the inputs with [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core), maps them to the arguments of the [`wp-release sign`](https://wpelevator.github.io/docs/js/wp-release/latest/signing.html) command and runs it through the `execute()` function of [`@wpelevator/wp-release`](https://www.npmjs.com/package/@wpelevator/wp-release), so the action and the CLI always behave the same. The action runs `dist/index.js`, a single file bundled with esbuild that includes `wp-release` and all other dependencies, so nothing is installed from npm when the action runs. The tooling version is fixed by the action ref you pin.

## Development

The source is TypeScript in `src/`. `dist/` is git-ignored in the monorepo. `npm run build` bundles the `@wpelevator/wp-release` dependency at the version that the action's `package.json` pins. The dependents policy of the monorepo release configuration re-releases the actions and updates their pins when `wp-release` is released, so they don't need a manual dependency update. The split-push workflow commits the bundle to the action's release repository. `npm test` runs the input mapping and action execution tests with Node.js type stripping, and `npm run lint` runs ESLint and a `tsc` type check.

```bash
npm run build --workspace @wpelevator/wp-release-sign-action
```
