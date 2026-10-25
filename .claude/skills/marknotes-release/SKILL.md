---
name: marknotes-release
description: Coordinate MarkNotes release readiness, artifact verification and manual downstream adoption without implicit publishing.
---

<!-- marknotes-workflow: dependency-handbook-v1 -->

# MarkNotes release coordination

This is the canonical common procedure. Read root `AGENTS.md`, the consuming repository's
`docs/dependencies.md`, and the canonical spec handbook's architecture, dependencies,
releasing and agents documents at the same revision as this body.

For consumer adapters, apply `docs/agents.md` provenance checks before following this body:
verify the actual pinned checkout and clean procedure files, or validate and record one
accepted-main SHA for all remote reads. A skill name/marker without those checks is not
proof of approved source. Stop on unresolved provenance.

## Establish authority and current state

Separate release-readiness research from permission to publish. Identify the exact
operations authorized by the user. Do not infer merge/tag/release/deploy/rollback
authority from invoking this skill or approving an implementation plan.

Record reviewed source SHA, version/tag resolved to commit, spec SHA, source versus
binary mode, dirty state, generator/toolchain versions and current downstream pins.
For Pro include the exact Libre source/catalog/nested pins and Pro root policy.
Check relevant pending work without treating it as shipped behavior.

## Check upstream readiness

Classify semantics, API, executor integration or build-only changes. For shared behavior,
require coordinated spec/fixtures/features and real reference/binding observations.
Require migration notes for new actions, fields, capabilities or durable-state behavior.

For sync, align Cargo/root npm/bindings npm versions and release tag. Validate both Git
source preparation and packed runtime consumption, matching Kotlin/native artifacts,
full release host/ABI coverage, archive structure and the exact checksum asset set.
Do not publish an empty native archive or claim same-origin hashes authenticate source.

For applications, use their own release/signing/provider/permission/hosting checks.
Do not equate an unsigned release-variant build with a signed distributable.
Preserve F-Droid source/reproducibility constraints and Pro provider/license separation.

## Coordinate consumers manually

Deliver a handoff containing source/tag/package identities, actual URLs/coordinates,
reviewed hashes and trust limits, spec pin, compiler/generator/JNA/ABI requirements,
API/executor migrations, validation results and responsible downstream owners.

Consumers update actual source/lock/catalog/gitlinks and verification policies.
An upstream release does not change their pins. Land reviewed Libre changes before Pro
adopts them; do not duplicate shared versions or wire helpers absent from the pinned tree.
Run app-owned provider/store/recovery and strict parity tests, not just library tests.

Do not create an update bot or downstream PR merely because this checklist mentions
follow-up work. Preserve manual ownership unless explicitly instructed otherwise.

## Report states separately

State readiness, publication, consumption, merge, application release and production
deployment independently. Web main merges can deploy even documentation changes, and a
Web GitHub release can precede a failed deploy. Desktop updater eligibility is not proof
of successful signed installation. Android recovery requires increasing version codes.

If an authorized release operation fails, report its real partial state and stop unsafe
follow-through. Never delete/recreate tags, auto-accept changed artifact digests, disable
checks or mutate production as a success-shaped fallback.

Return actual validation and explicit blockers/next owner. Do not claim unavailable
host, device, platform or production verification was performed.
