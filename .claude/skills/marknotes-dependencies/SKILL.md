---
name: marknotes-dependencies
description: Trace MarkNotes upstream/downstream dependencies and perform authorized consumption or validation changes.
---

<!-- marknotes-workflow: dependency-handbook-v1 -->

# MarkNotes dependency workflow

This is the canonical common procedure, owned by marknotes-spec. A consumer entrypoint
loads this body and supplies its own `docs/dependencies.md`; it must not recursively
load its adapter as the canonical skill.

## Establish scope and evidence

Read root `AGENTS.md` and the local dependency map. Read the canonical handbook's
`docs/architecture.md`, `docs/dependencies.md`, `docs/releasing.md` and `docs/agents.md`
from the same spec revision as this body. Paths are relative to that canonical source,
not necessarily the consumer's current directory.

Record the user's requested outcome: report-only, documentation, dependency update or
plumbing implementation. Do not turn investigation into an unrequested upgrade or release.
Record dirty state before edits; preserve unrelated changes.

When loaded through a consumer adapter, require the canonical provenance checks in
`docs/agents.md`: actual pinned checkout identity and clean procedure files, or one
validated, recorded accepted-main SHA for all remote procedure/handbook reads. A matching
skill name/marker alone does not authenticate its source. Stop on unresolved provenance.

Inventory the affected repository's current source revision and actual consumed
upstream/source/package/spec pins. For Pro inspect its pinned Libre tree/catalog and
nested pins, not only Libre main. Distinguish current main, pending work, release and
installed/deployed revision. Use immutable source evidence and state unresolved facts.

## Classify the edge and route ownership

Identify Git source, fixture submodule, released binary, generated binding, build tool
or app distribution. Find the manifest, lock/catalog, gitlink, resolver, generator and
verification policy that govern it.

Route shared semantics to spec first, shared algorithms/bindings to sync, executor/
provider/storage behavior to the app, shared Android modules/catalog to Libre, and
Pro host/provider adaptations to Pro. Do not copy algorithms or edit nested upstream
sources inside a downstream change.

Check known divergences and actual runner mappings. Never loosen expected fixture
values, return golden observations instead of execution or suppress unknown scenarios.

## Perform only an authorized change

For source npm dependencies, align intended full Git SHA, manifest, root lock and
resolved lock identity; update effective lifecycle approval where applicable.
Check the consumed source's compiler/generator requirements and run a real preparation/
install path. Lock-only generation is not a source build.

For Android release assets, align catalog/source pin, reviewed artifact identities and
active-root Gradle verification. Pro needs its own root policy. Do not replace trusted
hashes automatically. Distinguish bootstrap trust-on-first-use from authentication.

For local source mode, validate the actual selected checkout and generated outputs,
respect explicit path precedence and reject missing/stale outputs without a release
fallback. Explicit local experiments remain unreviewed; CI/signed releases require the
reviewed clean-source policy or the exact reviewed distributor non-build pruning profile,
not a broad dirty-tree bypass. Do not manufacture provenance by hashing arbitrary files.

For tooling/hosting, fix the owning source of truth and preserve existing checksum,
runtime, platform and job-name contracts. Do not add downstream update automation unless
the user explicitly requested it.

## Validate and hand off

Use local runbook commands and smallest relevant regression selectors first. Exercise
actual source generation/artifacts, binding/native loading, executor persistence and
strict parity when their boundaries change. Use isolated caches for tamper/offline tests.

Record changed consumption edges, actual commands/results, unrun gates, compatibility
requirements and the next manual owner. Separate implemented, merged, released, consumed
and deployed states. If an upstream merge is required but not authorized, report that
blocker rather than pinning an unmerged branch or committing a broken helper reference.

Stop before unrequested PR creation, merge, tag, publication, production deployment or
destructive rollback. A working dependency change is not permission to release it.
