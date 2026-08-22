# flux-d2-mirror

Mirrors third-party Helm charts into GHCR as OCI artifacts, so cluster sources are
uniformly `OCIRepository` rather than a mix of `HelmRepository` and OCI, and so the
cluster is insulated from upstream repositories disappearing or retagging.

Charts land at `ghcr.io/bendwyer/d2-mirror/charts/<name>:<version>`. Packages inherit
this repository's visibility, which is why it is public: private packages would require
node-level registry authentication, and containerd does not pick that up without a reboot.

## What is not obvious

`limit` defaults to `1`, so an entry without it mirrors only the highest version. That is
usually right, because a sync never prunes and earlier versions stay available once
mirrored. A fresh reseed, however, starts with no rollback history.

No `version` constraint is set on an entry. Version policy lives in Renovate, in the
repository that consumes the chart. Expressing it here as well would silently withhold a
new major from ever being seen.

Renovate tracks the mirror rather than upstream, so a version only becomes visible to it
once a sync has copied it. The schedule here therefore sets the update lag, and it also
removes any chance of a bump landing before the artifact exists.

A sync exits 2 on drift, meaning a tag already at the destination has a different digest
than the source. That fails the job deliberately: it means upstream republished a version
that should have been immutable.
