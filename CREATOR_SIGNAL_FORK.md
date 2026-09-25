# Creator Signal fork and release contract

This repository is an adopted fork of [minio/mc](https://github.com/minio/mc).
The upstream project remains the source for general MinIO Client development.
Creator Signal owns only the adoption line and the OCI release process described
here.

## Branches and ownership

- `master` tracks the GitHub fork's upstream default branch; do not make
  Creator Signal changes directly on it.
- `creator-signal/master` is the long-lived adopted integration line.
- Delivery work must use `issue/<Sales-Pulse-issue>-<slug>` and merge by pull
  request into `creator-signal/master`. Do not use convenience branch prefixes.
- Sales Pulse Issue #1888 owns the initial adoption and release pipeline.

Keep an `upstream` remote configured as `https://github.com/minio/mc.git`.
Before each reconciliation, fetch both remotes and inspect the change range:

```sh
git fetch origin --prune
git fetch upstream --tags --prune
git log --oneline origin/creator-signal/master..upstream/master
```

Create an issue branch from the current adoption line, merge or rebase the
reviewed upstream commit, resolve only documented Creator Signal overlay
conflicts, run the required checks, and open a PR into `creator-signal/master`.
Never force-push the adoption line. Record the upstream commit, conflict
resolution and release-impact assessment in the PR.

## OCI releases

The `Creator Signal OCI release` workflow runs only when an immutable Git tag
named `creator-signal-mc-vYYYYMMDD-<12-character-current-commit-sha>` is pushed.
It verifies that the tagged commit is an ancestor of `creator-signal/master` and
derives the matching OCI image tag
`cs-YYYYMMDD-<12-character-current-commit-sha>`. The Git tag is the explicit
release authorization; do not retag, move, or reuse it.

The workflow builds `Dockerfile.creator-signal` from the checked-out source;
it does not download an upstream `latest` binary. It publishes only immutable
tags to `ghcr.io/creator-signal/mc`, a content digest, an SBOM, provenance and
a vulnerability scan result. The published digest—not a mutable tag—is the
only approved input for a downstream Sales Pulse image-lock change.

A successful release is a candidate artifact. It does not authorize an
environment promotion or deployment. A separate Sales Pulse Issue/PR must
replace all current `quay.io/minio/mc` references and locks together, validate
the affected object-storage/recovery/observability paths including
`pnpm local:all`, and retain the old digest as the rollback target.
