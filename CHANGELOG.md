# Changelog

## 0.3.8 (2026-11-04)

- Bundle wp-release 0.20.1.

## 0.3.7 (2026-11-03)

- Bundle wp-release 0.20.0.

## 0.3.6 (2026-10-31)

- Bundle wp-release 0.19.0.

## 0.3.5 (2026-10-29)

- Bundle wp-release 0.18.2.

## 0.3.4 (2026-10-29)

- Bundle wp-release 0.18.1.

## 0.3.3 (2026-10-29)

- Bundle wp-release 0.18.0.

## 0.3.2 (2026-10-13)

- Bundle wp-release 0.17.0.

## 0.3.1 (2026-10-13)

- Make ZIP and manifest signing independent: --sign signs only ZIPs, --sign-checksums generates and signs manifests for ZIPs and directory builds, and --sign-format requires an explicit signing target. Update the ZIP Action and release examples. ([`e515a53`](https://github.com/wpelevator/basement/commit/e515a537ddb1b08a5b5839e81eb492f47f55911e))
- Expose checksum filenames, formats, embedding and configured signing overrides in the release actions. Separate embedded metadata outputs from uploadable files and align action examples, verification and CI instructions with the coordinated release. ([`4b3e411`](https://github.com/wpelevator/basement/commit/4b3e41112cb4c432f322441c72e87af11dd6a779))
- Bundle wp-release 0.16.0.

## 0.3.0 (2026-10-13)

- Add ZIP signing inputs and signature outputs, expose WordPress, minisign and SSH formats in the sign action, and update action documentation to match wp-release.
  
  Support configured SVN deployment with release-ref, package, list, config and tag-format inputs, preserve single-plugin deployment outputs, and expose configured plans and outcomes as JSON. Add the ZIP config input. Run the SVN action through the shared command with an injected runner that installs SVN only when needed. ([`ff25f51`](https://github.com/wpelevator/basement/commit/ff25f5188bcbfa03f98e8ce4dadf3324b79c1b84))
- Bundle wp-release 0.15.0.

## 0.2.7 (2026-10-13)

- Bundle wp-release 0.14.0.

## 0.2.6 (2026-10-12)

- Bundle wp-release 0.13.0.

## 0.2.5 (2026-10-12)

- Bundle wp-release 0.12.0.

## 0.2.4 (2026-10-12)

- Bundle wp-release 0.11.0.

## 0.2.3 (2026-10-12)

- Bundle wp-release 0.10.0.

## 0.2.2 (2026-10-11)

- Bundle wp-release 0.7.0.

## 0.2.1 (2026-10-11)

- Keep the release actions pinned to a stable wp-release version while the monorepo root resolves them to its local workspace; support explicit changelog insertion markers before the first version heading.

## 0.2.0 (2026-10-11)

- Each run adds a section to the job summary with the signature files and the public key. A failed run adds its error.
- Bundles `@wpelevator/wp-release` 0.5.0.

## 0.1.0 (2026-10-10)

- First release.
