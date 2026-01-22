# Secrets Comparison: release-secrets.md vs DOC-VARIANTS/02-per-system.md

## Analysis Summary

### ✅ Secrets Present in Both Documents

**Content Gateway (CGW)**:
- ✓ `cgw.cgwSecret` / `cgwSecret` / `CGW_USERNAME` + `CGW_PASSWORD` / `CGW_TOKEN`
- ✓ `content-gateway-service-account-read-only`

**Pyxis**:
- ✓ `pyxis.secret` / `pyxisSecret` / `PYXIS_CERT_PATH` + `PYXIS_KEY_PATH` / `PYXIS_GRAPHQL_API`

**Quay**:
- ✓ `mapping.registrySecret` / `registrySecret` / `publishingCredentials`
- ✓ All vault paths (including red-marked "not used" ones)
- ✓ `konflux-release-service-access-management`
- ✓ `redhat-workloads-token`
- ✓ `konflux-artifacts`

**GitLab**:
- ✓ `file_updates_secret` / `ACCESS_TOKEN`
- ✓ `advisory_secret_name`
- ✓ `managed-tenants` tokens
- ✓ `konflux-release-gitlab-bot` LDAP

**Pulp + UDC**:
- ✓ `PULP_SECRET_NAME` / `pulpSecret`
- ✓ `udcacheSecret`

**Exodus**:
- ✓ `exodusGwSecret` (certificates)
- ✓ `exodus-service-account`

**IIB / FBC**:
- ✓ `iibServiceAccountSecret`

**Errata**:
- ✓ `errata_secret_name`

**Slack**:
- ✓ `slack.slack-notification-secret` / `secretName` + `secretKeyName`

**GitHub**:
- ✓ `github.githubSecret` / `githubSecret` / `GITHUB_TOKEN`

**MRRC + RADAS**:
- ✓ `mrrc.awsSecret`
- ✓ `mrrc.signing.signCASecret`

**OOT kmods signing**:
- ✓ `ootsign.signing-secret` / `signing-secret`
- ✓ `ootsign.checksumFingerprint` / `checksumFingerprint`
- ✓ `ootsign.checksumKeytab` / `checksumKeytab`

**Koji**:
- ✓ `pushOptions.pushKeytab.secret` / `pushSecret`

**Git**:
- ✓ `git.tokenSecret` / `gitTokenSecret`

**Object storage**:
- ✓ `s3.credentialsSecret` / `s3CredentialsSecret`
- ✓ `azure.spSecret`

**JIRA**:
- ✓ `jira.jiraAdvisorySecret` / `jiraSecretName`

**Infra PR creator**:
- ✓ `infra.prCreatorSecret` / `sharedSecret`

**Atlas**:
- ✓ `atlas.atlas-sso-secret-name`
- ✓ `atlas.atlas-retry-aws-secret-name`

**Integration tests**:
- ✓ All test secrets (konflux-test-infra-secret, mapt-kind-secret, etc.)
- ✓ `redhat-appstudio-registry-pull-secret`
- ✓ `hacbs-release-tests-token`
- ✓ `redhat-appstudio-qe-bot-token`
- ✓ `release-tests-token`
- ✓ `test-cosign-secret`
- ✓ `file-updates-secret`
- ✓ `publish-to-cgw-secret`
- ✓ `exodus-prod-secret`

**ExternalSecrets**:
- ✓ `e2e-test-vault-password-secret`
- ✓ `e2e-test-service-account-kubeconfig`
- ✓ `e2e-test-github-token`
- ✓ `release-team-slack-webhook-notification-secret`

**Additional infrastructure**:
- ✓ `staging/release/registry-io-pull`
- ✓ `konflux/release/registry.redhat.io`
- ✓ UMB certificates and service accounts
- ✓ OSIDB service account
- ✓ Signing server service account
- ✓ Kubeconfig files (remote-client-config)

---

## ⚠️ Secrets in release-secrets.md NOT in 02-per-system.md

### 1. **Cosign Secret (Production)**
**In release-secrets.md**:
- Line 119: `sign.cosignSecretName` | Cosign AWS and signing key | `tasks/managed/rh-sign-image-cosign` | `secrets/konflux/release/cosign` | Signing service

**Status in 02-per-system.md**:
- Only mentioned as `test-cosign-secret` in Integration tests section (line 242)
- NOT listed as a production secret with its own section or under a signing/cosign section

**Impact**: Missing production cosign secret documentation

---

### 2. **Cloud Marketplaces Secret**
**In release-secrets.md**:
- Line 124: `mapping.cloudMarketplacesSecret` | Cloud marketplaces credentials | `tasks/managed/marketplacesvm-push-disk-images` | TBD | Cloud marketplace team
- Line 152: `cloudMarketplacesSecret` | `tasks/managed/marketplacesvm-push-disk-images` | TBD | Cloud marketplace owner

**Status in 02-per-system.md**:
- NOT found anywhere in the document
- No section for cloud marketplaces

**Impact**: Missing cloud marketplaces secret documentation

---

## Recommendation

Add the following to `DOC-VARIANTS/02-per-system.md`:

### 1. New section: "Cosign / Image Signing"
Should include:
- `sign.cosignSecretName` for production image signing
- Vault path: `secrets/konflux/release/cosign`
- Used in: `tasks/managed/rh-sign-image-cosign`
- Owner: Signing service

### 2. New section: "Cloud Marketplaces"
Should include:
- `mapping.cloudMarketplacesSecret` / `cloudMarketplacesSecret`
- Used in: `tasks/managed/marketplacesvm-push-disk-images`
- Vault path: TBD
- Owner: Cloud marketplace team
- Rotation: TBD

---

## Conclusion

**UPDATE**: The 2 missing secrets have been added to `02-per-system.md`:
1. ✅ Production Cosign secret (`sign.cosignSecretName`) - Added under "Cosign / Image Signing" section
2. ✅ Cloud Marketplaces secret (`mapping.cloudMarketplacesSecret`) - Added under "Cloud Marketplaces" section

**UPDATE 2**: The flowchart section has been added to `02-per-system.md`:
3. ✅ Flowchart: Secret Lifecycle and Design Patterns - Added with all 5 design patterns and key observations

**All content from release-secrets.md is now present in the per-system document.**
