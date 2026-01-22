# Release Secrets Comparison (Catalog Doc vs CSV)

This is a quick comparison between `release-secrets.md` (catalog-driven inventory) and `Release-Service-Secrets.csv` (vault/app-interface oriented inventory).

## High-confidence overlaps

These appear in both sources, though names/paths differ:
- **IIB service account**: CSV `staging/production/release/iib/iib-service-account` ↔ doc `iibServiceAccountSecret` / `fbc` publishing credentials context.
- **File updates GitLab token**: CSV `*/gitlab.cee/file-updates-secret` ↔ doc `file_updates_secret`.
- **CGW credentials**: CSV `publish-to-cgw-secret` / content-gateway service account ↔ doc `cgwSecret` and CGW env vars in `release-service-utils`.
- **Pyxis cert/key**: CSV `konflux/release/pyxis/*` ↔ doc `pyxis.secret` / `PYXIS_CERT_PATH` + `PYXIS_KEY_PATH`.
- **Pulp + UDC**: CSV `rhsm-pulp/*` and `udcache/*` ↔ doc `pulpSecret` / `udcacheSecret` and utilities referencing Pulp/UDC.
- **Exodus**: CSV `exodus/*` ↔ doc `exodusGwSecret`.
- **E2E secrets**: CSV `vault-password`, `e2e-test-service-account-kubeconfig`, `e2e-test-github-token` ↔ doc integration-test + konflux-release-data ExternalSecrets.
- **Quay tokens**: CSV multiple quay robot tokens ↔ doc `publishingCredentials`, `registrySecret`, and Quay-related usage.

## CSV-only (not explicitly called out in `release-secrets.md`)

These are detailed vault/app-interface secrets and operational accounts not yet reflected in the catalog-oriented doc:
- **Managed tenants tokens**: `*/gitlab.cee/managed-tenants*` (stage/prod).
- **Splunk forwarder**: `staging/release/logging/splunk-application-forwarder`.
- **LDAP bot**: `production/release/ldap/konflux-release-gitlab-bot`.
- **UMB**: `konflux/release/umb/*` service accounts and certs.
- **OSIDB service account**: `konflux/release/internal-services/osidb/prod`.
- **Signing server SA**: `konflux/release/internal-services/signing-servers/prod/konflux-release-signing-sa`.
- **Release-service bot GitHub account**: `konflux/release/github/release-service-bot`.
- **Redhat-workloads token**: `konflux/release/quay/prod/redhat-workloads-token`.
- **Remote-client kubeconfigs**: `hacbs-internal-services/internal-services-controller/remote-client-config/*`.
- **Registry.redhat.io pull secret**: `konflux/release/registry.redhat.io`.
- **Explicit vault paths + rotation SOPs** for many Quay robots.

## Doc-only (not in CSV)

These appear in `release-secrets.md` but are not listed in the CSV:
- **Atlas secrets** (`atlas-*`), **Azure SP**, **S3 credentials**, **OOT kmods signing secrets**.
- **JIRA advisory secret**, **GitHub release token**, **Infra PR creator secret** (GitHub App).
- **MRRC secrets** (AWS + RADAS signing CA).
- **Slack notification secret** (catalog task); CSV only shows release-team Slack webhook ExternalSecret in `konflux-release-data`.
- **Koji push keytab** (`pushOptions.pushKeytab.secret`), unless mapped in CSV as separate vault paths.

## Key structural differences

- **CSV is vault- and rotation-centric**: includes explicit Vault paths, expiry dates, and operational rotation steps.
- **Catalog doc is usage-centric**: focuses on where secrets are referenced in pipelines/tasks and code, with evidence paths.
- **Namespace mapping**: CSV implies app-interface/ExternalSecret wiring; the doc does not list a canonical per-env mapping unless it exists in `konflux-release-data`.

## Suggested next steps

- Add a **"Vault path / ExternalSecret name"** column to the catalog doc for items that map directly to CSV.
- Identify **which CSV entries are non-release-service** (e.g., platform/infra-only) and label them as “external dependency”.
- If desired, split CSV items by **env (staging/prod)** and align them to release pipelines or tenants in `konflux-release-data`.
