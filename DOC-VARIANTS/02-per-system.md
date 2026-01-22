# Release Service Secrets Inventory

> **Related JIRA**: [RELEASE-2070](https://issues.redhat.com/browse/RELEASE-2070) - Create comprehensive doc and flow-chart related to release Secrets

This variant groups secrets by external system or functional area to reduce table width and improve readability.

**Scope**: Covers secrets from `release-service-catalog`, `release-service-utils`, `internal-services`, `release-service`, and `konflux-release-data` repositories.

**Format note**: Secrets are organized by the external system they authenticate to, with rotation procedures placed immediately after each system's table.

---

**Common ServiceNow forms** (used in many rotation procedures):
- General support / password reset: https://redhat.service-now.com/help?id=sc_cat_item&sys_id=ec649e491bc74d587f9bfc8f034bcbe3
- Service catalog: https://redhat.service-now.com/help?id=rh_service_catalog

---

## Flowchart: Secret Lifecycle and Design Patterns

This flowchart visualizes how secrets flow from storage to consumption across different systems.

```mermaid
flowchart TD
    subgraph Storage["<b>Secret Storage</b>"]
        direction LR
        V1[Vault devshift<br/>konflux/*, releng/*]
        V2[Ansible Vault<br/>encrypted files]
    end

    subgraph Definitions["<b>CR Definitions</b>"]
        direction LR
        KRD[konflux-release-data<br/>ExternalSecret CRs]
        CAT[catalog repo<br/>vault/*.yaml]
    end

    subgraph CRSync["<b>CR Sync</b>"]
        direction LR
        ARGO[ArgoCD<br/>ApplicationSet]
        MANUAL[kubectl apply]
    end

    subgraph SecretSync["<b>Secret Sync</b>"]
        direction LR
        ESO[External Secrets<br/>Operator]
        DECRYPT[ansible-vault<br/>decrypt]
    end

    subgraph K8s["<b>K8s Secrets</b>"]
        direction LR
        K1[rhtap-releng-tenant]
        K2[rhtap-release-*-tenant]
        K3[Test namespaces]
    end

    subgraph Consume["<b>Release Service</b>"]
        direction LR
        RPA[ReleasePlanAdmission<br/>ReleasePlan]
        TASK[Tekton Tasks<br/>Pipelines]
    end

    subgraph Systems["<b>External Systems</b>"]
        direction LR
        SYS1["Pyxis • CGW • Quay • GitLab"]
        SYS2["Koji/IIB • Pulp/UDC/Exodus"]
        SYS3["GitHub • Slack/JIRA"]
    end

    Storage ~~~ Definitions
    Definitions ~~~ CRSync
    CRSync ~~~ SecretSync
    SecretSync ~~~ K8s
    K8s ~~~ Consume
    Consume ~~~ Systems
    
    KRD --> ARGO
    CAT --> MANUAL
    
    ARGO --> ESO
    MANUAL --> DECRYPT
    
    V1 --> ESO
    V2 --> DECRYPT
    
    ESO --> K1 & K2
    DECRYPT --> K3
    
    K1 & K2 & K3 --> TASK
    RPA --> TASK
    TASK --> SYS1 & SYS2 & SYS3

    style Storage fill:#e1f5ff
    style Definitions fill:#fff9e0
    style CRSync fill:#ffe0f0
    style SecretSync fill:#fff3e0
    style K8s fill:#f3e5f5
    style Consume fill:#e8f5e9
    style Systems fill:#ffebee
```

### Design Patterns Identified

1. **Managed secrets via ArgoCD pattern** (blue → yellow → pink → orange → purple → green):
   - Vault (`konflux/*` paths) → ExternalSecret CR in `konflux-release-data` → ArgoCD syncs CR to cluster → ESO fetches from Vault → K8s Secret (managed namespace) → ReleasePlanAdmission → Task
   - Used for: Pyxis, CGW, Pulp, IIB, advisory/errata
   - Managed by: Release Service team, credentials owned by external service teams
   - SecretStore: ClusterSecretStore `appsre-stonesoup-vault`

2. **Tenant secrets via ArgoCD pattern**:
   - Vault (`releng/*` paths) → ExternalSecret CR in `konflux-release-data` → ArgoCD syncs CR to cluster → ESO fetches from Vault → K8s Secret (tenant namespace) → ReleasePlan → Task
   - Used for: Slack notifications, e2e test credentials
   - Managed by: Release Service team, tenant configures usage in ReleasePlan
   - SecretStore: SecretStore `releng-vault`

3. **Integration test secrets pattern**:
   - Ansible Vault encrypted files in catalog repo → Manual `kubectl apply` → ansible-vault decrypt → K8s Secret (test namespace) → Test Pipeline
   - Used for: E2E tests, integration tests
   - Managed by: Konflux Releng team

4. **Multi-environment pattern**:
   - Same secret name, different Vault paths per environment (stage/prod/qa)
   - ExternalSecret CRs in different cluster directories in `konflux-release-data`
   - Used for: Pyxis, CGW, Pulp, UDC, advisory, IIB
   - Managed via environment-specific ArgoCD Applications

5. **Hybrid auth pattern**:
   - Single Vault path contains multiple auth credentials (username/password + token)
   - Example: CGW has both `CGW_USERNAME`/`CGW_PASSWORD` and `CGW_TOKEN`
   - Tasks consume based on API endpoint requirements

### Key Observations

- **GitOps-driven secret management**: Most secrets follow a GitOps pattern where ExternalSecret CRs in `konflux-release-data` are synced by ArgoCD, then ESO fetches actual secrets from Vault
- **Credential ownership fragmentation**: Credentials owned by different teams (Pyxis, CGW, Quay, GitLab, etc.) requiring coordinated rotation with Release Service
- **Mixed expiry models**: Some secrets expire (certs), some don't (tokens), some are rotated on-demand
- **Namespace complexity**: Secrets span multiple namespaces (managed vs tenant) with different access patterns and SecretStore configurations
- **Two-stage sync**: Secrets require both CR sync (ArgoCD) and secret value sync (ESO) before becoming available to Release Service

---

## Content Gateway (CGW)
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `cgw.cgwSecret` / `cgwSecret` / `CGW_USERNAME` + `CGW_PASSWORD` / `CGW_TOKEN` | `tasks/managed/create-advisory`, `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images`, `release-service-utils` CGW scripts | `konflux/release/internal-services/content-gateway/prod/content-gateway-service-account` | No expiry | CGW team | release-service-catalog<br>release-service-utils |

**Rotation procedure**:  
Generate new token from webUI at: https://source.redhat.com/groups/public/dxp/exd_digital_experience_platforms_dxp_blog/user_tokens_now_available_in_content_gateway

## Pyxis
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `pyxis.secret` / `pyxisSecret` / `PYXIS_CERT_PATH` + `PYXIS_KEY_PATH` / `PYXIS_GRAPHQL_API` | `tasks/managed/*pyxis*`, `release-service-utils/pyxis/*` | `konflux/release/pyxis/stage`, `konflux/release/pyxis/prod` | January 17, 2027 | Pyxis team | release-service-catalog<br>release-service-utils |

**Rotation procedure** (JIRA: RHTAPREL-894):
1. See Pyxis access request procedure at https://issues.redhat.com/browse/RHTAPREL-627
2. Use `service-account-credentials` found as key in Vault
3. Once new certificate is created, add it to Vault as new version
4. Propagate to RemoteSecrets (see "Adding a Secret to a Managed Workspace" guide)
5. Revoke the old certificate:
   - Determine the serial number: `openssl x509 -in hacbs-release-pyxis.crt -noout -text | grep 'Serial Number'`
   - Note the item in brackets (e.g., 0xffe15d7)
   - Go to https://redhat.service-now.com/help?id=rh_service_catalog and click "Request General Support"
   - Select IT, "Is there a service degradation?" → No, "Select the app" → RHCS, "Are you requesting something new?" → New
   - Enter body: "Please revoke the following certificates.\n- 0xffe15d7 (← change this)"

**Note**: When the Service Account name is changed, you must notify the CLOUDWF team via a ticket (see example: CLOUDWF-10575)

## Quay (registry credentials)
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `mapping.registrySecret` / `registrySecret` / `publishingCredentials` | `tasks/managed/make-repo-public`, FBC publishing pipelines | staging/release/quay/hacbs-release-tests_iib-pub<br>production/release/quay/rh-osbs__iib-pub<br>production/release/quay/rh-osbs__rhtap_preview_iib<br>production/release/fbc/publishing-credentials<br>production/release/fbc/publishing-credentials-redhat-prod<br>hacbs-internal-services/fbc/preview/publishing-credentials | No expiry | Quay admin (Tim Waugh) | release-service-catalog |

**Rotation procedure** (JIRA: RHTAPREL-685, RHTAPREL-694-702, etc.):  
Contact Tim Waugh to request rotation using the "RHTAP - SP Robot Rotation" process.

**Alternative for specific tokens**:
- For `hacbs-release-tests_iib-pub`: Create new robot account in Quay (add increasing suffix)
- For managed by team: `swickers`, `bhills`, or `pkhander` can update/rotate directly from Quay org

**FBC publishing credentials** (JIRA: RHTAPREL-700 prod, RHTAPREL-698 preview):  
Contains `sourceIndexCredential` and `targetIndexCredential` for multiple Quay robots. Contact Tim Waugh for rotation.

## GitLab (file updates + advisory)
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `file_updates_secret` / `ACCESS_TOKEN` | `pipelines/internal/process-file-updates`, `release-service-utils` git helpers | `staging/release/gitlab/gitlab.cee/file-updates-secret`, `production/release/gitlab/gitlab.cee/file-updates-secret` | No expiry | GitLab admin | release-service-catalog<br>release-service-utils |
| `advisory_secret_name` | `pipelines/internal/create-advisory`, `pipelines/internal/filter-already-released-advisory-images` | `konflux/release/internal-services/gitlab/gitlab.cee/create-advisory-prod-secret`, `konflux/release/internal-services/gitlab/gitlab.cee/create-advisory-stage-secret` | January 13, 2027 | GitLab admin | release-service-catalog |
| `managed-tenants` tokens | MRs in managed-tenants repos | `staging/release/gitlab/gitlab.cee/managed-tenants`, `production/release/gitlab/gitlab.cee/managed-tenants` | No expiry | GitLab admin | Internal automation |
| `konflux-release-gitlab-bot` LDAP | GitLab bot service account | `production/release/ldap/konflux-release-gitlab-bot` | No expiry | LDAP admin | Internal automation |

**Rotation procedure for file-updates tokens** (JIRA: RHTAPREL-682 stage, RHTAPREL-687 prod):
- **Staging** (`hacbs-release-tests/app-interface`):
  1. Go to https://gitlab.cee.redhat.com/hacbs-release-tests/app-interface/-/settings/access_tokens
  2. Note the Scopes
  3. Create new token with required scopes
- **Production** (`rhtap-release/app-interface`):
  1. Go to https://gitlab.cee.redhat.com/rhtap-release/app-interface/-/settings/access_tokens
  2. Note the Scopes
  3. Create new token with required scopes

**Rotation procedure for managed-tenants tokens** (JIRA: RHTAPREL-682 stage, new prod):
- **Staging** (`rhtap-release/managed-tenants-stage`):
  1. Go to https://gitlab.cee.redhat.com/groups/rhtap-release/-/settings/access_tokens
  2. Find the `stage-konflux-release-addons` token
  3. Note the Scopes
  4. Click rotate and copy the secret
- **Production** (`service/managed-tenants`):
  1. Login using LDAP account `konflux-release-gitlab-bot`
  2. Go to https://gitlab.cee.redhat.com/-/user_settings/personal_access_tokens
  3. Note the Scopes
  4. Create new token with required scopes

**Rotation procedure for advisory tokens**:
- **Production** (`releng/advisories`):
  1. Go to https://gitlab.cee.redhat.com/releng/advisories/-/settings/access_tokens
  2. Create new token with name `release-service-token-MMDDYYYY`, Developer role, scopes: `read_repository` + `write_repository`
- **Staging** (`rhtap-release/advisories`):
  1. Go to https://gitlab.cee.redhat.com/rhtap-release/advisories/-/settings/access_tokens
  2. Create new token with name `release-service-token-MMDDYYYY`, Developer role, scopes: `read_repository` + `write_repository`

**Rotation procedure for LDAP bot**:  
Submit password reset via ServiceNow: https://redhat.service-now.com/help?id=sc_cat_item&sys_id=ec649e491bc74d587f9bfc8f034bcbe3  
**Purpose**: Used as GitLab bot. Login using the username/password and generate access tokens

## Pulp + UDC
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `PULP_SECRET_NAME` / `pulpSecret` | `tasks/managed/push-rpms-to-pulp`, `pipelines/internal/push-artifacts-to-cdn` | `konflux/release/internal-services/rhsm-pulp/prod/certificate`, `konflux/release/internal-services/rhsm-pulp/stage/certificate`, `konflux/release/internal-services/rhsm-pulp/qa/certificate` | Jun 11, 2026 | Pulp team | release-service-catalog |
| `udcacheSecret` | `pipelines/internal/push-artifacts-to-cdn` | `konflux/release/internal-services/udcache/prod/certificate`, `konflux/release/internal-services/udcache/stage/certificate`, `konflux/release/internal-services/udcache/qa/certificate` | Jun 11, 2026 | UDC team | release-service-catalog |

**Rotation procedure for Pulp** (JIRA: RHELDST-25044):  
Follow the rotation steps detailed in https://issues.redhat.com/browse/RHELDST-25044  
**Contains**: cert, key, and url for accessing pulp in the pulp_push_wrapper script

**Rotation procedure for UDC** (JIRA: TEAMNADO-7368):  
Follow the rotation steps detailed in https://issues.redhat.com/browse/TEAMNADO-7368  
**Contains**: cert, key, and url for flushing udcache instance in the pulp_push_wrapper script

## Exodus (CDN)
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `exodusGwSecret` (certificates) | `pipelines/internal/push-artifacts-to-cdn`, `pipelines/internal/push-disk-images` | `konflux/release/internal-services/exodus/prod/certificates`, `konflux/release/internal-services/exodus/nonprod/certificates` | March 29, 2026 | Exodus GW team | release-service-catalog |
| `exodus-service-account` | Service account for exodus CDN | `konflux/release/internal-services/exodus/prod/exodus-service-account`, `konflux/release/internal-services/exodus/nonprod/exodus-service-account` | No expiry | Exodus GW team | Internal automation |

**Rotation procedure for certificates** (JIRA: CLOUDDST-22187 prod, CLOUDDST-22185 nonprod):
- **Production**: Follow https://issues.redhat.com/browse/CLOUDDST-22187
- **Nonprod**: Follow https://issues.redhat.com/browse/CLOUDDST-22185

**Rotation procedure for service accounts** (JIRA: CLOUDDST-22185):  
Follow the rotation steps detailed in https://issues.redhat.com/browse/CLOUDDST-22185

## IIB / FBC service account
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `iibServiceAccountSecret` | `pipelines/internal/update-fbc-catalog`, `pipelines/internal/check-fbc-opt-in`, `tasks/managed/add-fbc-contribution` | `staging/release/iib/iib-service-account`, `production/release/iib/iib-service-account` | No expiry | IIB team | release-service-catalog |

**Rotation procedure** (JIRA: RHTAPREL-685 stage, new prod):
1. Request new ServiceAccount (new numbered suffix) and keytab via ServiceNow: https://redhat.service-now.com/help?id=sc_cat_item&sys_id=ec649e491bc74d587f9bfc8f034bcbe3
2. Create MR to update this section in the catalog
3. Once MR is merged, update app-interface with a MR
4. Then create new ServiceNow ticket (same form) asking to delete old Service Account

## Errata
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `errata_secret_name` | `pipelines/internal/create-advisory` | `konflux/release/internal-services/errata/prod/errata-service-account`, `konflux/release/internal-services/errata/stage/errata-service-account` | No expiry | Errata team | release-service-catalog |

**Rotation procedure**:  
Submit password reset via ServiceNow: https://redhat.service-now.com/help?id=sc_cat_item&sys_id=ec649e491bc74d587f9bfc8f034bcbe3  
**Purpose**: Used for obtaining a Kerberos ticket as our errata service account and communicating with the errata API

## Slack
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `slack.slack-notification-secret` / `secretName` + `secretKeyName` | `tasks/managed/send-slack-notification` | `releng/konflux/rhtap-releng-tenant/common/release-team-slack-webhook-notification-secret` | No expiry | Slack admin | release-service-catalog |

**Rotation procedure**: TBD

## GitHub
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `github.githubSecret` / `githubSecret` / `GITHUB_TOKEN` | `tasks/managed/create-github-release`, `release-service-utils` promote overlay | `konflux/release/github/release-service-bot` | No expiry | GitHub admin | release-service-catalog<br>release-service-utils |

**Rotation procedure**: TBD  
**Contains**: email, password, and recovery keys for https://github.com/release-service-bot  
**Purpose**: Used in catalog promotion script and internal-services final pipeline

## Cosign / Image Signing
| Secret | Where used | Vault path(s) | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- | --- |
| `sign.cosignSecretName` | `tasks/managed/rh-sign-image-cosign` | `secrets/konflux/release/cosign` | No expiry | Signing service | release-service-catalog |

**Rotation procedure**: TBD  
**Purpose**: Cosign AWS and signing key for image signing

## Integration tests (release-service-catalog)

These secrets are used by e2e and integration test pipelines in the catalog:

| Secret param/name | Purpose | Vault path(s) | Expiry | Credential Owner |
| --- | --- | --- | --- | --- |
| `redhat-appstudio-registry-pull-secret` | Pull images from docker.io/quay.io in build PipelineRun | Default after cluster provision | No expiry | Konflux Releng |
| `hacbs-release-tests-token` | Pull/push images from quay.io in managed PipelineRun | staging: dockerconfigjson for hacbs-release-tests+m5_robot_account<br>ephemeral: QUAY_TOKEN env | No expiry | Konflux Releng |
| `redhat-appstudio-qe-bot-token` | Create releases in GitHub repos | GITHUB_TOKEN env | No expiry | Konflux Releng |
| `release-tests-token` | Push images to Pyxis staging repo | redhat-pending+rhtap_release_integration2 from Vault | No expiry | Konflux Releng |
| `test-cosign-secret` | Used by rh_sign_cosign task | `secrets/konflux/release/cosign` (yuzheng provided, jluza provided PUBLIC_KEY) | No expiry | Konflux Releng |
| `file-updates-secret` | Used by run-file-updates task | Token for https://gitlab.cee.redhat.com/hacbs-qe/app-interface | No expiry | Konflux Releng |
| `publish-to-cgw-secret` | Used by publish-to-cgw task | `secrets/konflux/release/internal-services/content-gateway` (created by pkhander/swickers) | No expiry | Konflux Releng |
| `exodus-prod-secret` | Used by push-to-cdn task | `secrets/konflux/release/internal-services/exodus/prod/certificates` (created by pkhander/swickers) | No expiry | Konflux Releng |

**Rotation procedure**: TBD (managed by Konflux Releng team)

## ExternalSecrets (konflux-release-data)
| ExternalSecret | Vault key | Namespace | SecretStore | Expiry | Credential Owner |
| --- | --- | --- | --- | --- | --- |
| `e2e-test-vault-password-secret` | `stonesoup/staging/release/e2e/vault-password` | `rhtap-release-2-tenant` | `appsre-stonesoup-vault` (ClusterSecretStore) | No expiry | Konflux Releng |
| `e2e-test-service-account-kubeconfig` | `stonesoup/staging/release/e2e/e2e-test-service-account-kubeconfig` | `rhtap-release-2-tenant` | `appsre-stonesoup-vault` (ClusterSecretStore) | No expiry | Konflux Releng |
| `e2e-test-github-token` | `stonesoup/staging/release/e2e/e2e-base-github-token` | `rhtap-release-2-tenant` | `appsre-stonesoup-vault` (ClusterSecretStore) | No expiry | Konflux Releng |
| `release-team-slack-webhook-notification-secret` | `releng/konflux/rhtap-releng-tenant/common/release-team-slack-webhook-notification-secret` | `rhtap-releng-tenant` | `releng-vault` (SecretStore) | No expiry | Konflux Releng |

**Rotation procedure**: TBD (managed via ExternalSecrets operator and Vault)

## Additional infrastructure secrets

Secrets documented in the CSV but not directly referenced in catalog tasks (used by internal services, automation, or infrastructure):

| Vault path | Purpose | Expiry | Credential Owner | Repository |
| --- | --- | --- | --- | --- |
| `staging/release/registry-io-pull` | Pull secret for registry.redhat.io | No expiry | Registry admin | Internal automation |
| `konflux/release/registry.redhat.io` | Pull secret for building release-utils in Konflux | No expiry | Registry admin | release-service-utils |
| `stonesoup/production/release/quay/konflux-release-service-access-management` | Quay app for repo access management | No expiry | Quay admin (Ralph Bean) | Internal automation |
| `konflux/release/quay/prod/redhat-workloads-token` | Pull token for konflux-ci/redhat-workloads | No expiry | Quay admin (Ralph Bean) | internal-services |
| `konflux/release/quay/prod/konflux-artifacts` | Sharing OCI artifacts between Konflux and signing hosts | No expiry | Quay admin (swickers/bhills/pkhander) | Internal automation |
| `konflux/release/umb/nonprod/umb-certificates` | UMB nonprod crt and key files | January 8, 2027 | UMB team | Internal automation |
| `konflux/release/umb/nonprod/umb-service-account` | UMB nonprod service account (id, password) | No expiry | UMB team | Internal automation |
| `konflux/release/umb/prod/umb-certificates` | UMB prod crt and key files | January 8, 2027 | UMB team | Internal automation |
| `konflux/release/umb/prod/umb-service-account` | UMB prod service account (id, password) | No expiry | UMB team | Internal automation |
| `konflux/release/internal-services/content-gateway/prod/content-gateway-service-account-read-only` | CGW read-only prod service account | No expiry | CGW team | Internal automation |
| `konflux/release/internal-services/osidb/prod/osidb-service-account` | OSIDB Kerberos service account | No expiry | OSIDB team | Internal automation |
| `konflux/release/internal-services/signing-servers/prod/konflux-release-signing-sa` | Service account for publishing signed binaries to developer portal | No expiry | Signing team | Internal automation |
| `hacbs/hacbs-internal-services/internal-services-controller/remote-client-config/hacbs-dev` | Kubeconfig for hacbs-dev cluster | No expiry | Cluster admin | release-service-utils |
| `hacbs/hacbs-internal-services/internal-services-controller/remote-client-config/staging-p01` | Kubeconfig for staging-p01 cluster | No expiry | Cluster admin | release-service-utils |
| `hacbs/hacbs-internal-services/internal-services-controller/remote-client-config/stonesoup-prod` | Kubeconfig for stonesoup-prod cluster | No expiry | Cluster admin | release-service-utils |

**Rotation procedures**:

### registry.redhat.io pull secrets (JIRA: RHTAPREL-686)
- **For staging** (`staging/release/registry-io-pull`):
  1. Regenerate token at https://access.redhat.com/terms-based-registry/token/stonesoup
  2. Go to the "Openshift Secret" tab
  3. Copy the `.dockerconfigjson` value
  4. Update Vault
- **For production** (`konflux/release/registry.redhat.io`):
  Generate new password at https://access.redhat.com/terms-based-registry/#/

### UMB certificates
Follow the UMB Client Guide to generate new client cert. **Note**: UMB service accounts (id and password) do NOT require rotation.

### OSIDB service account
Submit password reset via ServiceNow: https://redhat.service-now.com/help?id=sc_cat_item&sys_id=ec649e491bc74d587f9bfc8f034bcbe3  
**Purpose**: Used for obtaining a Kerberos ticket as our osidb service account and communicating with the osidb API

### Signing server service account (JIRA: CLDX-77)
Submit a ServiceNow request to IT asking to rotate the password  
**Purpose**: Service account for publishing signed binaries to developer portal

### Kubeconfig files (JIRA: RHTAPREL-690, RHTAPREL-707)
1. Re-run `get-kubeconfig-from-service-account.sh`
2. Copy file to clipboard
3. Create new version in Vault


## Out of scope - managed by other teams

The following secrets have incomplete information (Vault path and/or expiry set to TBD) and are managed by other teams:

| Name | Secret | Where used | Credential Owner | Repository |
| --- | --- | --- | --- | --- |
| MRRC AWS | `mrrc.awsSecret` | `tasks/managed/publish-to-mrrc` | MRRC team | release-service-catalog |
| OOT kmods signing | `ootsign.signing-secret` / `signing-secret` | `tasks/managed/sign-oot-kmods` | Signing team | release-service-catalog |
| OOT kmods checksum fingerprint | `ootsign.checksumFingerprint` / `checksumFingerprint` | `tasks/managed/sign-oot-kmods` | Signing team | release-service-catalog |
| OOT kmods checksum keytab | `ootsign.checksumKeytab` / `checksumKeytab` | `tasks/managed/sign-oot-kmods` | Signing team | release-service-catalog |
| Koji push keytab | `pushOptions.pushKeytab.secret` / `pushSecret` | `tasks/managed/push-rpm-to-koji`, `tasks/managed/promote-koji-draft-build`, `tasks/managed/update-package-collection` | Koji team | release-service-catalog |
| Git token (OOT kmods) | `git.tokenSecret` / `gitTokenSecret` | `tasks/managed/push-oot-kmods-to-git` | Git host admin | release-service-catalog |
| S3 credentials | `s3.credentialsSecret` / `s3CredentialsSecret` | `tasks/managed/push-oot-kmods-to-s3` | S3 owner | release-service-catalog |
| Azure Service Principal | `azure.spSecret` | Tasks not in this repo (placeholder) | Azure owner | External (TBD) |
| JIRA Advisory | `jira.jiraAdvisorySecret` / `jiraSecretName` | `tasks/managed/populate-release-notes` | JIRA admin | release-service-catalog |
| Infra PR Creator | `infra.prCreatorSecret` / `sharedSecret` | `tasks/managed/update-infra-deployments` | infra-deployments owner | release-service-catalog |
| Atlas SSO | `atlas.atlas-sso-secret-name` | `tasks/managed/collect-atlas-params` | Atlas team | release-service-catalog |
| Atlas Retry AWS | `atlas.atlas-retry-aws-secret-name` | `tasks/managed/collect-atlas-params` | Atlas team | release-service-catalog |
| Cloud Marketplaces | `mapping.cloudMarketplacesSecret` / `cloudMarketplacesSecret` | `tasks/managed/marketplacesvm-push-disk-images` | Cloud marketplace team | release-service-catalog |
| Konflux test infra (internal-services) | `konflux-test-infra-secret` / `cloud-credential-key` / `cluster-access-secret` / `remote-cluster-access-secret` / `local-cluster-access-secret` / `mapt-kind-secret` | `internal-services` e2e pipeline | Konflux Releng | internal-services |
| Konflux test infra (release-service) | `konflux-test-infra-secret` / `mapt-kind-secret` / `cluster-access-secret-name` / `konflux-e2e-secrets` | `release-service` e2e pipeline | Konflux Releng | release-service |

**Note**: These secrets are referenced in the catalog but lack complete documentation. They are managed by other teams and are out of scope for Release Team secret management.

---

## Data sources

This document was compiled from:
1. **Code analysis**: `release-service-catalog`, `release-service-utils`, `internal-services`, `release-service` repositories
2. **Schema**: `schema/dataKeys.json` for canonical data keys
3. **Configuration**: `konflux-release-data` repository for ExternalSecret definitions
4. **CSV inventory**: `Release-Service-Secrets.csv` for Vault paths, expiry dates, rotation procedures, and JIRA trackers
5. **Documentation**: Integration test READMEs and maintenance guides

