<!-- marknotes-workflow: dependency-handbook-v1 -->

# Dependency consumption

This document explains dependency identities, build channels and verification boundaries.
Use the [architecture map](architecture.md) to locate ownership and the
[release runbook](releasing.md) to coordinate a change.

## Identity is not one version number

Record these independently:

| Identity | Example at the audit | Where to read it |
| --- | --- | --- |
| Shared contract revision | A spec commit | Consumer's `spec` gitlink; Pro's `libre/spec` |
| Sync source revision | `48bdbdeda451963154d3a8defb7efc0d9a6f4cbb` | Git dependency/lock or Android's sync gitlink |
| Sync package version | `0.4.0` | Cargo workspace and both upstream npm manifests; consumer catalog/lock |
| Shared Android source revision | Libre `7479889`, tagged `v0.1.4` | Pro's `libre` gitlink |
| App distribution version | Pro `v0.4.0` at the audit | App release workflow/tag/assets, not the sync version |
| Build-tool versions | Rust/bindgen, Gradle/JDK, Node/npm | Owner's toolchain files, wrappers, actions and build image |
| Deployment revision | Web `GIT_SHA` and `APP_VERSION` | Build metadata, deployment result and `/health` |

Equal numeric versions across apps/libraries are coincidental unless explicitly stated.
An app's manifest version is not necessarily its latest release version: some release
workflows override it only in the build checkout.

## Edge-by-edge consumption

| Consumer | Consumed item/channel | Pin surfaces | Update owner and verification |
| --- | --- | --- | --- |
| Sync | Spec source/fixtures | `.gitmodules`, `spec` gitlink | Sync maintainer; owned fixture mappings, reference executor and binding parity |
| Libre | Spec source/fixtures | `.gitmodules`, `spec` gitlink | Libre maintainer; strict Android parity, including shared UI tests |
| Libre | Published sync JAR/AAR through Ivy | `gradle/libs.versions.toml`, source gitlink; root verification policy when adopted | Libre maintainer; tag/source alignment, Gradle artifacts, JVM/device and executor checks |
| Pro | Libre shared source/catalog | `.gitmodules`, `libre` gitlink; `settings.gradle.kts` project/catalog wiring | Pro maintainer after Libre lands; API/catalog compatibility, strict parity, APK permissions |
| Pro | Sync assets and spec through Libre | Imported `libs`, nested gitlinks; Pro root resolver/verification | Pro maintainer; active-root verification and Pro host-native/provider integration |
| Desktop | Sync Git source npm package | `package.json`, `package-lock.json`, source-based script approval | Desktop maintainer; source preparation, exact SHA consistency, real-core/process/package checks |
| Web | Sync Git source npm package | `assets/package.json`, `assets/package-lock.json` | Web maintainer; source preparation, real-core/browser/CSP/offline/container checks |
| Desktop/Web | Spec source/fixtures | `.gitmodules`, `spec` gitlink | Respective maintainer; complete mappings and strict app-key observations |

Source provenance pins do not replace byte verification of separately downloaded assets.
Fixture pins do not advance when a package changes, or vice versa.

## Sync outputs and channels

Sync is not published to npm, Maven Central or GitHub Packages at the audited revision.
Its npm manifests are private; its Rust crates are unpublished.

| Output | Consumption | Important distinction |
| --- | --- | --- |
| `marknotes-sync-X.Y.Z.tgz` | Released `@marknotes/sync` npm package | Already embeds generated WASM; does not need Rust at consumer install |
| Root Git npm package | `github:MaxDac/marknotes-sync#<full-SHA>` or exact HTTPS Git URL | npm preparation builds the package from source |
| `marknotes-sync-kotlin-X.Y.Z.jar` | Generated Kotlin API classes | No Android native payload |
| `marknotes-sync-natives-X.Y.Z.jar` | Host native libraries for JVM tests | JNA resource prefixes; not an Android substitute |
| `marknotes-sync-android-X.Y.Z.aar` | Android native libraries | Complementary to Kotlin JAR, not a replacement |
| `SHA256SUMS` | Release byte-consistency evidence | Not an independent publisher signature |

The upstream root and `bindings/npm` npm manifests and Cargo workspace must describe
the same version. Generated Kotlin archive versions derive from Cargo.
Release filenames and scoped npm package names are different identities.

### Network and credential boundaries

| Channel | Used for | Configuration owner |
| --- | --- | --- |
| GitHub HTTPS source/releases | Spec/Libre gitlinks, sync Git source, released sync assets | Consumer Git/npm/Ivy settings and reviewed pins |
| npm registries and lock-recorded feeds | JavaScript dependencies and build tools | Each manifest/lock and consumer registry policy |
| Cargo registry and Rust distribution | Rust crates, compiler/target and binding CLI | Sync Cargo lock/toolchain; consumer source-build setup |
| Maven Central, Google Maven and plugin portal | JVM/Android libraries, plugins and build tooling | Active Gradle root and shared/Pro catalogs |
| Restricted Microsoft feed | Pro's required Microsoft transitive dependency | Pro root repository content filter |
| Container registries | Web build/runtime base images | Dockerfile and image-update review |

Public sync assets are not a private package registry and do not require npm or GitHub
Packages credentials. Source builds still need access to their compiler/crate/tool
channels. Authenticated maintainer API/publication operations, CI signing/deployment
credentials and end-user cloud-provider sign-in are separate concerns.
Document required variable names/scopes locally, never their secret values.

Inspect actual lockfile `resolved` URLs and integrity algorithms rather than assuming
one registry policy covers all repositories. At the audited sync tree, binding-tooling
entries in `bindings/npm/package-lock.json` use `ms-feed-25.pkgs.visualstudio.com` and
SHA-1 integrity metadata; the app locks have their own policies/origins. This is neither
an npm publication of sync nor evidence of SHA-512 verification for those tooling inputs.
Record provenance concerns separately; do not silently rewrite locks during documentation work.

### WASM source preparation

At the audited sync revision, root `prepare` runs `bindings/npm/build.mjs`.
It builds with locked Cargo dependencies and the WASM feature, then runs the pinned
wasm-bindgen CLI. The wrapper embeds generated WASM bytes and initializes synchronously
on import; apps do not configure a separate sync `.wasm` download URL.

Web/Desktop source installs require Git, a host C/C++ linker, Rust `1.99.1`,
`wasm32-unknown-unknown`, and wasm-bindgen CLI `0.2.129`. These are snapshot values:
check the consumed sync revision before changing them. A lock-only npm operation does
not build bindings, and disabled lifecycle scripts do not produce a usable source install.

Core use extends beyond pressing Sync: local metadata/hash/merge helpers can import it.
Verify Node tests, Desktop main/renderer and browser bundling where applicable.
Do not confuse sync WASM with optional PDF import WASM or worker assets.

At this revision, the package resets its WASM instance after a `WebAssembly.RuntimeError`
and rethrows the failure; ordinary argument errors are not silently converted into success.
Scheduler handles from the previous instance cannot be reused. Preserve explicit handle
cleanup and application failure handling when changing imports or scheduler lifecycles.

### Kotlin/native generation

The Kotlin namespace is `io.github.maxdac.marknotes.sync`; native library name is
`marknotes_sync`. Generated bindings and native payloads must match.
The upstream workspace-pinned UniFFI generator is used, not an arbitrary installed CLI.

Android outputs contain `arm64-v8a` and `x86_64` native libraries, minimum API 29,
with 16 KiB alignment checks. Host-native packaging covers Linux x86-64, Windows x86-64
and macOS arm64/x86-64 in a full release. Packaging an archive without generating/staging
its real native inputs first is not a successful native build.

At the audit, source rebuilding pins NDK `30.0.16248370` and cargo-ndk `4.1.2`.
JDK 17 is a binding bytecode target; application CI uses JDK 21. These are not
interchangeable requirements. Windows source rebuilding needs the appropriate native
linker/toolchain and Git Bash; use the owner's documented setup.

## Android and Pro

Artifact-only Ivy maps the group `io.github.maxdac.marknotes` to:

```text
https://github.com/MaxDac/marknotes-sync/releases/download/v[revision]/[module]-[revision].[ext]
```

Coordinates are:

```text
io.github.maxdac.marknotes:marknotes-sync-kotlin:X.Y.Z
io.github.maxdac.marknotes:marknotes-sync-natives:X.Y.Z
io.github.maxdac.marknotes:marknotes-sync-android:X.Y.Z@aar
```

There is no POM to supply transitive runtime dependencies. JNA must be explicit
(`net.java.dev.jna:jna`, `5.19.1` at the audit; Android selects its AAR).
`:core` exposes Kotlin bindings and supplies host natives/JNA to JVM tests.
`:app-shared` supplies the Android native AAR and Android JNA.
Pro's `:pro-core` tests also need the host-native path.

### Published versus local source mode

Normal mode resolves released assets. `-Pmarknotes.sync.fromSource=<checkout>` selects
already-built artifacts under that checkout's `bindings/kotlin/build/libs`.
An active optional development-pin file can select the nested checkout by default.
The explicit override takes precedence; relative paths belong to the invoking build
root, including Pro, not an arbitrary shell working directory.

Source selection is not source compilation. Run the owner's source builder and validate
the selected checkout's actual identity/version; another commit with the same Cargo
version is not equivalent. Do not silently fall back to published assets if local
outputs are missing. Published digests cannot generally match locally rebuilt output:
host-native archive contents and toolchain/output state can differ.

The rollout's Android helper revalidates source/build receipts on every source-mode
invocation and deliberately rejects Gradle configuration cache in that mode.
Only the three selected source artifacts use changing-dependency semantics; other
verification stays active, and switching back to published mode restores their
published checksum checks even with the same cache.

F-Droid builds use pinned source and offline-vendored Cargo inputs rather than shipping
downloaded sync binaries. Preserve that path and its independent reproducibility checks.
Pro's non-free provider dependency chain is not automatically F-Droid eligible merely
because shared modules come from Libre.

### Root verification and adoption

Gradle verification metadata is owned by the active build root. Adding it under
`libre/gradle` does not protect Pro. A Pro adoption must bring its own root policy and
check its sync subset against the pinned Libre policy; shared digests must not become
independently maintained version pins.

The audit found no checked-in sync artifact checksums in the Android/Pro roots.
This rollout introduces that protection; do not assume a consumer is protected until
its adoption is merged and its real resolution paths are exercised. Check current
local runbooks for implementation status and exact commands.

Published verification should reject changed/missing artifacts, absent or stale hash
entries, and catalog/policy disagreement, including warm/offline cache use.
Source mode requires a different explicit contract, not global verification-off.
Local experiments must be identified as unreviewed; they cannot satisfy protected CI
or signed-release policy. F-Droid's reviewed recipe can intentionally remove tracked
non-build files: an exact checked-in deletion-only pruning profile must distinguish
that transformation from arbitrary dirty source. Modifications, renames and extra build
inputs are not covered by such a profile. Receipts must bind the actual allowed source
state and successful generation, not merely hash arbitrary existing outputs.
The rollout's Libre-owned profile is `gradle/marknotes-sync-fdroid-prune.txt`, checked
against the source-relative removal entries in `.fdroid.yml`. It is not a command-line
or environment-variable switch to accept arbitrary pruning; Pro obtains the policy only
when it adopts the corresponding Libre source revision.

## Web and Desktop install/build chains

Desktop uses the root npm manifest/lock; Web uses `assets/`. Both consumed the exact
same sync source SHA at the audit, but update them independently.
Their source-build toolchains are duplicated in consumer setup actions/scripts/images
and must be checked against that consumed upstream revision.

Desktop's dependency policy also includes compatibility overrides and a checked patch.
Remove them only when their upstream parent no longer requires them and packaging/
updater tests pass. npm advisory checks do not audit all Rust dependencies.
An npm script-approval table is version-sensitive and is not a sandbox for commands
explicitly executed by root `postinstall`.

Web's image builds in Rust/Node stages and runs only Caddy/static output.
Production Caddy and test Caddy should derive from the same supported version source;
the installer retains checksum verification before extraction. Service-worker caching,
content security policy and emitted worker/assets are part of dependency compatibility.

## Important third-party families

| Area | Examples and owners | What an upgrade can affect |
| --- | --- | --- |
| Rust/bindings | Cargo/serde/sha2, UniFFI, wasm-bindgen; sync | Public shapes, generator ABI, compiler/CLI alignment, native outputs |
| Android UI/storage | Kotlin/AGP/Compose, Room, coroutines; Libre catalog | Pro compilation, exported aliases, database/UI behavior, merged APK |
| Android Pro providers | Proton SDK plus compatible Proton core, MSAL, OkHttp; Pro | Native ABIs, login/redirect registrations, merged permissions, SDK interoperability |
| Desktop providers/runtime | Electron, Proton Drive SDK/crypto, MSAL Node; Desktop | Main-process bundling, credential flows, installers, native runtime and updater |
| Browser providers | MSAL browser, direct Dropbox/Graph adapters; Web | Redirect origins, scopes, browser fetch restrictions and conditional semantics |
| Editor/rendering/import | CodeMirror/Markdown renderers, sanitizers, PDF/DOCX conversion; app | Untrusted content handling, workers/assets, offline import and bundle size |
| Persistence/security | IndexedDB/idb/Web Crypto, Android Keystore, OS keychain; app | Migration, transactions, lock/key lifecycle and recovery |
| Build/test/distribution | Node/npm, Vite/TypeScript, Gradle/JDK/NDK, Playwright/Vitest, Caddy, electron-builder/updater | Lock formats, source preparation, platform behavior, hosting and delivery |

Do not equate `devDependency` with "never shipped": bundlers can include its runtime
code or assets. Web has intentional TypeScript package aliases; inspect their purpose
before simplifying them. Branding generation is a separate maintenance tool surface.
SDK licenses and native/transitive provenance need review, not assumptions.

## Verification and trust

| Evidence | Establishes | Does not establish |
| --- | --- | --- |
| Full Git SHA plus reviewed source | Intended source identity | Bytes/provenance of an independently downloaded binary |
| npm lock/integrity | Declared resolution and integrity where npm actually enforces it | Git-source/prepared-output verification, equality of duplicated Git pins or Rust advisory coverage |
| Release `SHA256SUMS` | Byte consistency against that manifest | Independent authentication if payload and checksum share a compromised origin |
| Reviewed checked-in Gradle hashes | Subsequent bytes match the reviewed baseline | Independent authenticity of initial trust-on-first-use bootstrap |
| Build receipt/output hashes | Outputs still match a recorded build state | Source authenticity if merely generated from arbitrary existing outputs |
| Parity/adapter tests | Tested contract behavior on those paths | Real provider consent/setup or every device/platform/race |
| Reproducible build comparison | Compared outputs match for tested inputs/environment | A universal guarantee for all later releases/toolchains |

The initial release-hash bootstrap is explicitly reviewed trust-on-first-use using the
current GitHub release and its checksum manifest. Never auto-regenerate trusted metadata
just because verification fails. Inspect origin, release/source identity and unexpected
replacement first. Keep tampering tests in temporary repositories/caches.

npm can report that it skips integrity checking for a Git dependency, even when its lock
entry contains an integrity field. Do not treat that field as proof of verification of
the cloned source or generated package. Record actual install behavior and the reviewed
full source identity; released tarballs and Git source have different trust boundaries.

## Troubleshooting by boundary

| Symptom | Check first | Do not do |
| --- | --- | --- |
| npm Git dependency fails during preparation | Exact source SHA, lifecycle permissions, Rust target/CLI/linker | Add an app-owned sync fallback or pretend a lock-only install built WASM |
| Kotlin compiles but native loading fails | Matching bindings/native JAR or AAR, JNA, host prefix/device ABI | Substitute a JVM natives JAR for Android or package empty native outputs |
| Source override loads stale output | Actual selected checkout, build state/receipt, version and output digests | Trust a matching version string or fetch release bytes silently |
| Artifact hash mismatch | Reviewed metadata, source/release tuple, cached/downloaded bytes | Disable verification or automatically accept new hashes |
| Missing/skipped parity | Initialize correct nested spec; run required parity task | Treat ordinary unit success as parity or loosen `expected` |
| Pro cannot find shared aliases/modules | Exact `libre` checkout, imported catalog and root project paths | Duplicate shared versions or edit nested sources in a Pro change |
| Pro login redirect fails | Provider registration, application identity and signing hash | Commit secrets or conflate debug and release redirect configuration |
| Web build passes but hosting fails | Selected Caddy, MIME/CSP/cache contracts, generated assets | Assume a Vite-only check proves production behavior |

Use each repository's `docs/dependencies.md`, README and test guide for executable local
commands. An unavailable tool or fixture is a prerequisite failure, not a passing result.

## Audited snapshot

The following immutable trees establish the initial documentation baseline. Follow their
manifests, locks, settings, workflows and local guides; refresh evidence for later work.

| Repository | Source tree |
| --- | --- |
| Spec | [82518019180953ddaead0ecb013f5231b5d187ac](https://github.com/MaxDac/marknotes-spec/tree/82518019180953ddaead0ecb013f5231b5d187ac) |
| Sync | [48bdbdeda451963154d3a8defb7efc0d9a6f4cbb](https://github.com/MaxDac/marknotes-sync/tree/48bdbdeda451963154d3a8defb7efc0d9a6f4cbb) |
| Libre main | [eafa1af7c8113da94e198dac18b22ee3662642d5](https://github.com/MaxDac/marknotes-android/tree/eafa1af7c8113da94e198dac18b22ee3662642d5) |
| Pro | [d7c1fc9a544a99521a021ae986c7f50d052fbc8e](https://github.com/MaxDac/MarkNotes/tree/d7c1fc9a544a99521a021ae986c7f50d052fbc8e) |
| Pro's Libre | [7479889462e5c3b21e5aa98ca0ae3dee279c2ea4](https://github.com/MaxDac/marknotes-android/tree/7479889462e5c3b21e5aa98ca0ae3dee279c2ea4) |
| Desktop | [50911b9a8a63a8049f409174abfeb0edf0a6156c](https://github.com/MaxDac/marknotes-electron/tree/50911b9a8a63a8049f409174abfeb0edf0a6156c) |
| Web | [6bb1f881bae6adde4e57ff83fee0347177c84f98](https://github.com/MaxDac/marknotes-web/tree/6bb1f881bae6adde4e57ff83fee0347177c84f98) |

At that snapshot, sync pinned spec `f265a5c`; Libre main pinned `12875c7`; Web,
Desktop and Pro's Libre pinned `0f92fac`. Those differences are meaningful consumption
evidence, not permission to advance all consumers without running their checks.
