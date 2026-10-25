# Dependency updates and release coordination

This runbook coordinates owners; it is not permission to merge, tag, publish or deploy.
Start with the [ownership map](architecture.md) and [consumption matrix](dependencies.md).
Repository-local `docs/dependencies.md`, README and test guides provide executable commands.

## Classify before changing pins

| Class | Required sequence |
| --- | --- |
| Shared behavior/format | Agree spec/fixtures/features; implement sync algorithms and relevant app executor behavior; advance consumers with mappings/tests |
| Sync API or generated binding | Update Rust and both bindings/types/tests; validate source/package outputs; release if binary consumers need it; update actual consuming pins |
| Sync source-only fix | Test the exact source revision; update Web/Desktop source pins and locks as needed; do not pretend Android consumes that source by default |
| Shared Android change | Land Libre shared implementation/catalog; review Pro API/provider/build impact; advance Pro's Libre gitlink |
| Provider/app-only fix | Update owning app; check shared executor contract and parity impact; track other apps only if their behavior is affected |
| Build/distribution validation | Repair owning tool/policy and regression tests; preserve existing release/deployment authorization and job names |
| Documentation/skills | Update common procedures in spec and local maps in apps; do not advance old fixture pins merely to deliver prose |

Spec-first is required for a shared contract change, not for every private implementation
detail. Never loosen `expected` to match one consumer. A divergence belongs in the shared
fixture annotation and feature matrix until the implementation is verified.

## Capture an immutable inventory

For every affected repository, record its default-branch SHA, actual consumed upstream
SHA/version, spec gitlink, source/binary mode, local changes and relevant pending work.
For Pro inspect the tree at its Libre gitlink, not Libre's current main.

Useful read-only commands include:

```sh
git status --short
git rev-parse HEAD
git ls-tree HEAD spec
git ls-tree HEAD libre
gh api repos/MaxDac/marknotes-sync/commits/main --jq .sha
gh release view v0.4.0 --repo MaxDac/marknotes-sync --json tagName,assets,targetCommitish
```

The version is an example from the [audit](dependencies.md#audited-snapshot), not a
permanent latest pin. Resolve a tag to its actual commit, including annotated-tag peeling;
release `targetCommitish` alone can be a branch name. Never print secret configuration.

Distinguish current main, reviewed candidate, published release, consumer pin and deployed
revision in reports. A merged upstream change is not automatically consumed downstream.

## Coordinating sync changes

1. Establish whether semantics, API, executor integration or only build plumbing changes.
   For semantics, land the coordinated spec update and map every affected fixture.
2. Implement upstream Rust and public Kotlin/WASM/npm surfaces together. Verify generated
   bindings, handwritten public types, boundary tests and parity observations.
3. Check all three version manifests: Cargo workspace, root npm and `bindings/npm`.
   Release tags and explicit artifact versions must agree; retain prerelease suffixes.
4. Exercise the source install path separately from the packed release path. A usable
   Git package needs its source helper files during preparation; a release tarball must
   contain its generated runtime output without needing Rust at install time.
5. Assemble the Kotlin JAR, host-native JAR, Android AAR and npm tarball with real inputs.
   Run structural/archive/native checks, full-host checks for releases, and complete
   digest validation. Publish only with explicit authorization.
6. Handoff source SHA, version/tag, artifact identities/digests, spec revision, required
   compiler/generator versions and API/executor migration requirements to consumers.
7. Consumers update their actual channel: Web/Desktop Git manifest+lock and script policy
   where applicable; Libre catalog+source pin+reviewed hashes or explicit development pin.
   Regenerate with supported tooling; do not manually fabricate lock integrity.
8. Run consumer-owned executor/provider/store tests as well as core fixtures. Handle new
   action/record fields and durable migrations, not merely successful compilation.
9. Land Libre changes before advancing Pro. Review the full consumed Libre delta and
   Pro root policy, imported catalog, host-native tests, ABI and APK permissions.

Sync `0.4.0`, for example, added preserved-remote-folder actions/state and notice/recreation
handling. Updating a dependency alone was insufficient: app executors and durable stores
needed the migration. Consult the release/source contract for later versions.

An app can intentionally consume reviewed source ahead of a binary release for development.
Record that status explicitly. Signed/publication paths must not treat an unreviewed local
experiment as a published, verified release.

## Consumer acceptance evidence

| Consumer | Required evidence for a sync/consumption change |
| --- | --- |
| Sync | Version consistency, Rust/core and binding boundary/parity tests, package smoke, archive/native/checksum validation |
| Libre | Catalog/source/policy consistency, selected-source identity when applicable, Gradle verification, JVM/native/executor tests, strict parity and Android native smoke |
| Pro | Reviewed Libre gitlink, root verification equality/enforcement, shared API/provider compatibility, `:pro-core` host natives, strict parity, APK permissions and native integration |
| Desktop | Manifest/root-lock/resolved SHA plus effective script approval, real fresh install, types/unit/parity, main/renderer and packaged-app checks |
| Web | Manifest/lock source identity, build-tool alignment, real-core/executor/parity, Caddy hosting/CSP/worker/offline and applicable container/browser checks |

Use the smallest relevant tests first, then the existing integration gates.
Ordinary Android unit tasks can skip missing fixtures; use the dedicated required parity
commands in the local runbook. A green divergence assertion means the known difference
still exists, not that the implementation now matches the canonical result.

CI job execution and protected-branch requirements are different inventories. Check live
rules/settings before asserting what blocks a merge, and do not rename required jobs.

## Verification failure handling

Stop and identify the failed boundary. Check the selected source/tag, catalog/lock,
reviewed metadata, generated output and cache identity. Test suspect bytes only in an
isolated cache or fixture, not by corrupting a shared developer cache.

Do not disable verification, auto-accept replacement checksums, drop a fixture mapping,
catch an unknown executor scenario as a divergence, or report stale generated output as
a successful rebuild. Same-origin release checksums establish consistency against that
manifest, not independent publisher authentication.

The initial checked-in release hashes use explicitly reviewed trust-on-first-use.
Record the bootstrap release and evidence. Subsequent changed bytes require a reviewed
update, not a silent refresh. Local source-mode evidence remains distinct from those
published-asset hashes.

## Release and deployment are different products

| Repository | Distribution at the audit | Operational distinction |
| --- | --- | --- |
| Sync | Tag-triggered GitHub Release with four packages and SHA256SUMS | Not npm/Maven publication; CI/artifact validation precede upload |
| Libre | Prepare-release PR flow, then manual checked/signed APK release; F-Droid source rebuilding | Application tag/version and reproducibility/signing checks are independent of sync's version |
| Pro | Manual APK release with signing/provider guards and version overrides | Consumes a Libre source pin; no shared-module package publication |
| Desktop | Manual platform installers, update metadata/blockmaps, checksums | AppImage/NSIS self-update; deb manual; macOS successful installation requires signing/notarization |
| Web | Main-push production deployment; manual tag/release plus deployment | GitHub release can succeed before deployment fails; health revision is separate acceptance |

No inspected repository supplied an automatic, coordinated sync-to-all-consumers update
workflow. Dependabot updates selected ecosystems; immutable Git commits, semver metadata
conditions and nested pins must be inspected rather than assumed to advance automatically.

Web main merges, including documentation-only changes, run checks and can deploy production.
Do not merge just to unblock a skill/document lookup without authorization.
Preserve service-worker/data/origin safeguards and deployment concurrency.

## Local experiments, release builds and recovery

Development can use explicitly selected local source. It must not silently download a
release instead, confuse a matching version with the reviewed commit, or reuse unrelated
old generated artifacts. CI/signed-release paths require reviewed source identity and
clean tracked sources, or an explicitly reviewed distributor pruning profile that
allows only the recipe's exact non-build deletions. It must not permit unrelated changes,
renames or extra build inputs. Record the actual transformed source in build evidence;
do not call it pristine or an unreviewed experiment.
An explicit local experiment must remain visibly unreviewed.

Unsigned release-variant compilation is a useful development/permission check; do not
confuse it with permission to publish a signed release.

F-Droid toolchain/recipe changes need distributor metadata coordination as well as app
code changes. Do not assume a repository-local recipe supersedes official fdroiddata.
Pro's provider SDK chain has separate licensing/distributor constraints.

For a bad release, identify source, artifacts and installed/deployed populations before
acting. Android must roll forward version codes; already-updated Desktop users do not
automatically downgrade when a release disappears. Web rollback must respect persisted
data and service-worker compatibility. Deleting tags/releases or changing production is
an explicitly authorized operation, never an automatic agent repair.

## Manual handoff record

For each affected edge, provide:

| Field | Required content |
| --- | --- |
| Owner and scope | Which upstream/downstream maintainers act; contract/API/build-only classification |
| Source identity | Reviewed commit, tag resolved to commit, source/binary/development mode |
| Consumption | Exact package URL/coordinate, manifests/locks/catalog/gitlinks changed |
| Artifacts and trust | Expected filenames, reviewed digests/policy, bootstrap or independent evidence, limitations |
| Build compatibility | Rust/bindgen or UniFFI/native/JNA/ABI requirements and generated outputs |
| Behavior | Spec SHA, changed mappings/actions/state, divergences and migrations |
| Validation | Commands actually executed and their outcomes, plus unrun/blocked gates |
| Completion state | Published upstream, consumed, merged, released and deployed as separate states |
| Next owner | Concrete manual action and prerequisite; do not silently create an update PR |

If an upstream helper must first land on main, leave its consumer integration explicitly
blocked. Do not pin a private feature branch or commit broken references to satisfy a
checklist. An approved implementation plan does not authorize the prerequisite merge.

## Tracking requested cross-repository work

All six repositories carry the `marknotes` topic and share the
[MarkNotes project](https://github.com/users/MaxDac/projects/16).
When the user requests issues or pull requests, follow local templates, add the resulting
items to that project and set `Repo area`: `spec`, `sync`, `android`, `desktop` or `web`.
Libre and Pro both use `android`; the repository identity still distinguishes them.

```sh
gh repo list MaxDac --topic marknotes
gh project item-add 16 --owner MaxDac --url <requested-issue-or-pr-url>
gh project field-list 16 --owner MaxDac --format json
```

Read current project field/option IDs before editing them; do not hard-code audit-time
IDs as permanent configuration. Link each dependent change and identify its manual next
owner. Project membership is tracking, not proof of merge/readiness or authority to create
unrequested items. Keep one repository's implementation in its own isolated worktree.
