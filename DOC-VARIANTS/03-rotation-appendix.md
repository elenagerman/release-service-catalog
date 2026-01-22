# Release Secrets (Variant 3: Rotation Appendix)

This variant keeps the main tables compact and moves rotation steps into an appendix.

## Canonical data keys (Release data)
| Data key | Purpose | Where used (examples) | Vault path(s) | Owner |
| --- | --- | --- | --- | --- |
| `cgw.cgwSecret` | Content Gateway auth (username/token) | `tasks/managed/create-advisory`, `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images` | `konflux/release/internal-services/content-gateway/prod/content-gateway-service-account` | Release Service (credential owner: Content Gateway team) |
| `jira.jiraAdvisorySecret` | JIRA token for advisory workflow | `tasks/managed/populate-release-notes` | TBD | Release Service (credential owner: JIRA admin) |
| `infra.prCreatorSecret` | GitHub App private key for infra PRs | `tasks/managed/update-infra-deployments` | TBD | Release Service (credential owner: infra-deployments repo admin) |
| `github.githubSecret` | GitHub token for releases | `tasks/managed/create-github-release` | `konflux/release/github/release-service-bot` | Release Service (credential owner: GitHub org admin) |
| `pyxis.secret` | Pyxis cert/key pair | `tasks/managed/*pyxis*` | `konflux/release/pyxis/stage`, `konflux/release/pyxis/prod` | Release Service (credential owner: Pyxis team) |
| `sign.cosignSecretName` | Cosign AWS and signing key | `tasks/managed/rh-sign-image-cosign` | `secrets/konflux/release/cosign` | Release Service (credential owner: signing service) |
| `slack.slack-notification-secret` | Slack webhook secret | `tasks/managed/send-slack-notification` | `releng/konflux/rhtap-releng-tenant/common/release-team-slack-webhook-notification-secret` | Release Service (credential owner: Slack admin) |
| `atlas.atlas-sso-secret-name` | Atlas SSO auth | Tasks not in this repo (placeholder) | TBD | Release Service (credential owner: Atlas team) |
| `atlas.atlas-retry-aws-secret-name` | Atlas retry S3 auth | Tasks not in this repo (placeholder) | TBD | Release Service (credential owner: Atlas team) |
| `mapping.registrySecret` | Quay.io API token | `tasks/managed/make-repo-public` | Known vault paths (from CSV):<br>staging/release/quay/hacbs-release-tests_iib-pub<br><span style="color: red;">staging/release/quay/redhat-dev+rhtap_release_integration</span><br><span style="color: red;">staging/release/quay/redhat-pending</span><br><span style="color: red;">staging/release/quay/redhat-pending+rhtap_release_integration</span><br><span style="color: red;">staging/release/quay/rh-osbs__iib-pub-pending</span><br><span style="color: red;">production/release/quay/redhat</span><br>production/release/quay/rh-osbs__iib-pub<br>production/release/quay/rh-osbs__rhtap_preview_iib<br><span style="color: red;">production/release/quay/rhtap_fbc_release_preprod</span> | Release Service (credential owner: Quay admin) |
| `mapping.cloudMarketplacesSecret` | Cloud marketplaces credentials | `tasks/managed/marketplacesvm-push-disk-images` | TBD | Release Service (credential owner: Cloud marketplace team) |
| `ootsign.signing-secret` | OOT kmods signing (signUser, signKey, signHost) | `tasks/managed/sign-oot-kmods` | TBD | Release Service (credential owner: signing team) |
| `ootsign.checksumFingerprint` | SSH host key DB for signing server | `tasks/managed/sign-oot-kmods` | TBD | Release Service (credential owner: signing team) |
| `ootsign.checksumKeytab` | Kerberos keytab for signing | `tasks/managed/sign-oot-kmods`, `tasks/managed/update-package-collection` | TBD | Release Service (credential owner: signing/Koji team) |
| `git.tokenSecret` | Git token for push | `tasks/managed/push-oot-kmods-to-git` | TBD | Release Service (credential owner: Git host admin) |
| `s3.credentialsSecret` | S3 credentials | `tasks/managed/push-oot-kmods-to-s3` | TBD | Release Service (credential owner: S3 owner) |
| `azure.spSecret` | Azure SP for blob storage | Tasks not in this repo (placeholder) | TBD | Release Service (credential owner: Azure owner) |
| `mrrc.awsSecret` | MRRC AWS credentials | `tasks/managed/publish-to-mrrc` | TBD | Release Service (credential owner: MRRC team) |
| `pushOptions.pushKeytab.secret` | Koji Kerberos keytab | `tasks/managed/push-rpm-to-koji`, `tasks/managed/promote-koji-draft-build`, `tasks/managed/update-package-collection` | TBD | Release Service (credential owner: Koji team) |

## Task and pipeline secrets (catalog)
| Secret param | Where used | Vault path(s) | Owner |
| --- | --- | --- | --- |
| `pyxisSecret` | `tasks/managed/*pyxis*` | `konflux/release/pyxis/stage`, `konflux/release/pyxis/prod` | Release Service / Pyxis team |
| `githubSecret` | `tasks/managed/create-github-release` | `konflux/release/github/release-service-bot` | Release Service / GitHub admin |
| `sharedSecret` | `tasks/managed/update-infra-deployments` | TBD | Release Service / infra-deployments owner |
| `jiraSecretName` | `tasks/managed/populate-release-notes` | TBD | Release Service / JIRA admin |
| `cgwSecret` | `tasks/managed/create-advisory`, `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images` | `konflux/release/internal-services/content-gateway/prod/content-gateway-service-account` | Release Service / CGW owner |
| `pushSecret` | `tasks/managed/push-rpm-to-koji`, `tasks/managed/promote-koji-draft-build`, `tasks/managed/update-package-collection` | TBD | Release Service / Koji team |
| `PULP_SECRET_NAME` | `tasks/managed/push-rpms-to-pulp` | `konflux/release/internal-services/rhsm-pulp/prod/certificate`, `konflux/release/internal-services/rhsm-pulp/stage/certificate`, `konflux/release/internal-services/rhsm-pulp/qa/certificate` | Release Service / Pulp team |
| `iibServiceAccountSecret` | `pipelines/internal/update-fbc-catalog`, `pipelines/internal/check-fbc-opt-in`, `tasks/managed/add-fbc-contribution` | `staging/release/iib/iib-service-account`, `production/release/iib/iib-service-account` | Release Service / IIB team |
| `publishingCredentials` | `pipelines/internal/*fbc*`, `tasks/internal/update-fbc-catalog-task` | `staging/release/quay/hacbs-release-tests_iib-pub`, `production/release/fbc/publishing-credentials`, `production/release/fbc/publishing-credentials-redhat-prod`, `hacbs-internal-services/fbc/preview/publishing-credentials` | Release Service / registry owner |
| `signing-secret` | `tasks/managed/sign-oot-kmods` | TBD | Release Service / signing team |
| `checksumFingerprint` | `tasks/managed/sign-oot-kmods` | TBD | Release Service / signing team |
| `checksumKeytab` | `tasks/managed/sign-oot-kmods` | TBD | Release Service / signing team |
| `s3CredentialsSecret` | `tasks/managed/push-oot-kmods-to-s3` | TBD | Release Service / S3 owner |
| `gitTokenSecret` | `tasks/managed/push-oot-kmods-to-git` | TBD | Release Service / Git host admin |
| `cloudMarketplacesSecret` | `tasks/managed/marketplacesvm-push-disk-images` | TBD | Release Service / cloud marketplace owner |
| `registrySecret` | `tasks/managed/make-repo-public` | Known vault paths (from CSV):<br>staging/release/quay/hacbs-release-tests_iib-pub<br><span style="color: red;">staging/release/quay/redhat-dev+rhtap_release_integration</span><br><span style="color: red;">staging/release/quay/redhat-pending</span><br><span style="color: red;">staging/release/quay/redhat-pending+rhtap_release_integration</span><br><span style="color: red;">staging/release/quay/rh-osbs__iib-pub-pending</span><br><span style="color: red;">production/release/quay/redhat</span><br>production/release/quay/rh-osbs__iib-pub<br>production/release/quay/rh-osbs__rhtap_preview_iib<br><span style="color: red;">production/release/quay/rhtap_fbc_release_preprod</span> | Release Service / Quay admin |
| `secretName` + `secretKeyName` | `tasks/managed/send-slack-notification` | `releng/konflux/rhtap-releng-tenant/common/release-team-slack-webhook-notification-secret` | Release Service / Slack admin |
| `file_updates_secret` | `pipelines/internal/process-file-updates` | `staging/release/gitlab/gitlab.cee/file-updates-secret`, `production/release/gitlab/gitlab.cee/file-updates-secret` | Release Service / Git host admin |
| `advisory_secret_name` | `pipelines/internal/create-advisory`, `pipelines/internal/filter-already-released-advisory-images` | `konflux/release/internal-services/gitlab/gitlab.cee/create-advisory-prod-secret`, `konflux/release/internal-services/gitlab/gitlab.cee/create-advisory-stage-secret` | Release Service / advisory owner |
| `errata_secret_name` | `pipelines/internal/create-advisory` | `konflux/release/internal-services/errata/prod/errata-service-account`, `konflux/release/internal-services/errata/stage/errata-service-account` | Release Service / errata owner |
| `exodusGwSecret` | `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images` | `konflux/release/internal-services/exodus/prod/certificates`, `konflux/release/internal-services/exodus/nonprod/certificates` | Release Service / Exodus GW owner |
| `pulpSecret` | `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images` | `konflux/release/internal-services/rhsm-pulp/prod/certificate`, `konflux/release/internal-services/rhsm-pulp/stage/certificate`, `konflux/release/internal-services/rhsm-pulp/qa/certificate` | Release Service / Pulp team |
| `udcacheSecret` | `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images` | `konflux/release/internal-services/udcache/prod/certificate`, `konflux/release/internal-services/udcache/stage/certificate`, `konflux/release/internal-services/udcache/qa/certificate` | Release Service / UDC owner |

## Related repos (quick inventory)
| Secret/env | Where used | Vault path(s) | Owner |
| --- | --- | --- | --- |
| `CGW_USERNAME` + `CGW_PASSWORD` | `release-service-utils` CGW wrappers | `konflux/release/internal-services/content-gateway/prod/content-gateway-service-account` | Release Service / CGW owner |
| `CGW_TOKEN` + `CGW_HOST` | `release-service-utils` CGW scripts | `konflux/release/internal-services/content-gateway/prod/content-gateway-service-account` | Release Service / CGW owner |
| `PYXIS_CERT_PATH` + `PYXIS_KEY_PATH` | `release-service-utils/pyxis/*` | `konflux/release/pyxis/stage`, `konflux/release/pyxis/prod` | Release Service / Pyxis team |
| `PYXIS_GRAPHQL_API` | `release-service-utils/pyxis/*` | `konflux/release/pyxis/stage`, `konflux/release/pyxis/prod` | Release Service / Pyxis team |
| `GITHUB_TOKEN` | `release-service-utils` promote overlay | `konflux/release/github/release-service-bot` | Release Service / GitHub admin |
| `ACCESS_TOKEN` | `release-service-utils` git helpers | `staging/release/gitlab/gitlab.cee/file-updates-secret`, `production/release/gitlab/gitlab.cee/file-updates-secret` | Release Service / GitLab admin |
| `internal-service-request-service-account-secret` | `release-service-utils` kubeconfig helper | `hacbs/hacbs-internal-services/internal-services-controller/remote-client-config/*` | Release Service / cluster admin |

## Integration tests
| Secret param/name | Where used | Vault path(s) | Owner |
| --- | --- | --- | --- |
| `konflux-test-infra-secret` / `cloud-credential-key` / `cluster-access-secret` / `remote-cluster-access-secret` / `local-cluster-access-secret` / `mapt-kind-secret` | `internal-services` e2e pipeline | TBD | Konflux Releng |
| `konflux-test-infra-secret` / `mapt-kind-secret` / `cluster-access-secret-name` / `konflux-e2e-secrets` | `release-service` e2e pipeline | TBD | Konflux Releng |

## ExternalSecrets (konflux-release-data)
| ExternalSecret | Vault key | Namespace | SecretStore | Owner |
| --- | --- | --- | --- | --- |
| `e2e-test-vault-password-secret` | `stonesoup/staging/release/e2e/vault-password` | `rhtap-release-2-tenant` | `appsre-stonesoup-vault` (ClusterSecretStore) | Konflux Releng / Release team |
| `e2e-test-service-account-kubeconfig` | `stonesoup/staging/release/e2e/e2e-test-service-account-kubeconfig` | `rhtap-release-2-tenant` | `appsre-stonesoup-vault` (ClusterSecretStore) | Konflux Releng / Release team |
| `e2e-test-github-token` | `stonesoup/staging/release/e2e/e2e-base-github-token` | `rhtap-release-2-tenant` | `appsre-stonesoup-vault` (ClusterSecretStore) | Konflux Releng / Release team |
| `release-team-slack-webhook-notification-secret` | `releng/konflux/rhtap-releng-tenant/common/release-team-slack-webhook-notification-secret` | `rhtap-releng-tenant` | `releng-vault` (SecretStore) | Konflux Releng / Release team |

## Appendix: Rotation details

Only entries with explicit rotation guidance are listed here.

| Secret | Rotation detail |
| --- | --- |
| Pyxis cert/key | Expires Jan 17 2027; rotate per RHTAPREL-894 |
| CGW service account | Rotate via CGW web UI token flow |
| Quay robot tokens | Rotate via RHTAP SP Robot Rotation |
| Pulp certs | notAfter Jun 11 2026; rotate per RHELDST-25044 |
| UDC certs | notAfter Jun 11 2026; rotate per TEAMNADO-7368 |
| Exodus certs | Expires Mar 29 2026; rotate per CLOUDDST-22185/22187 |
| IIB service account | Rotate via ServiceNow SA+keytab process |
| GitLab access tokens (file updates) | Rotate via GitLab access token procedure |
| Advisory GitLab tokens | Expires Jan 13 2027; rotate via GitLab token procedure |
| Errata service accounts | Rotate via ServiceNow errata SA reset |
