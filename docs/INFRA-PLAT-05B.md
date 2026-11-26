# INFRA-PLAT-05B - ServiceDatabases

## Objective

Validate the controlled continuation of the existing tenant provisioning
operation, creating isolated Identity and Clinical databases inside the
PostgreSQL instance provisioned during PLAT-05A.

## Baseline

- Infrastructure: a8bb77b7451939ffcc05f0748f68a0469891f598
- Platform: 3de5c3a5cd53dd42dfbf37298e23dbf5049aef00
- Platform PR: #20
- Platform CI: 45740315813
- Tenant: 01a214af-fddc-713e-a908-c1f690486d90
- Operation: 01a214af-fddc-75c8-9c5e-7f2317a4e6c2
- Cluster: pg-01a214affddc713ea908c1f690486d90
- Cluster UID: 84fb9b6c-b40a-4c85-b822-810907498f73

## Constraints

- Preserve the existing catalog, operation, Cluster and PVC.
- Preserve PLAT-05A manifests.
- No additional PostgreSQL instances.
- No credentials committed to Git.
- No manual SQL modifications to durable provisioning state.
- No additional operators.
- Worker remains at zero replicas until explicitly authorized.
- Resume through the internal Platform command.

## Application order

1. Validate all manifests without persistent changes.
2. Apply and verify admission policies.
3. Apply the pg_hba merge patch to the existing Cluster.
4. Wait for successful PostgreSQL reconciliation.
5. Create empty distribution Secrets.
6. Apply the additional RBAC and test effective permissions.
7. Import and verify the image built from the approved Platform commit.
8. Apply the Worker manifest with zero replicas.
9. Verify readiness prerequisites before controlled activation.
10. Resume and validate the existing operation.

Do not apply the entire directory recursively.
01-pg-hba.yaml is a merge patch, not a standalone resource.

Do not reapply empty distribution Secrets after credentials have been
distributed without first checking the resulting change.

## Acceptance criteria

- Existing Cluster UID and PVC preserved.
- Exactly two service DatabaseRoles and two Databases.
- Role creation starts with NOLOGIN.
- Roles remain non-privileged.
- Authentication and distribution Secrets match the authorized names.
- Both service owners can connect to their own databases using TLS.
- Cross-database connections are rejected.
- Unauthorized Kubernetes operations are rejected.
- The same durable operation is resumed without duplication.
- Reconciliation and retries preserve existing resources.
- No sensitive values appear in collected evidence.

## Status

INTEGRATION_PASS (2026-11-26).
LAB-10 remains postponed.

## Final integration evidence - 2026-11-26

**Verdict: INTEGRATION_PASS**, limited to ServiceDatabases.

### Integrated versions

- Infrastructure PR #16: 3f084368bb1efd8f896018756f4498286588f57f.
- Original Platform integration: 3de5c3a5cd53dd42dfbf37298e23dbf5049aef00.
- Platform FIX-2 PR #21.
- Audited correction: 0f992f49d25ea2e3e4478dff77a5379847d8b05c.
- Integrated correction: c8041c145e2292665cef67e5c848239bd95ad3aa.
- Platform post-merge CI: 45776954150 (SUCCESS).
- Platform correction suite: 187/187 tests reported passing.
- CloudNativePG: 1.30.2.
- Corrected local image configuration digest:
  sha256:a915212ea4086268c7487b80666e2e3aa5761db5e243d98507de79685a2a589b.

### Security preparation

- 32/32 effective RBAC checks passed.
- 22/22 admission dry-run tests passed.
- All four additional admission policies type-checked without warnings.
- Required pg_hba rules installed and independently verified.
- FIX-2 continuity: 17 manifest resources without drift.
- 10/10 representative effective permissions reconfirmed.

### Serialization incident and recovery

Attempt 2 stopped with FAILED_TERMINAL / CNPG_ROLE_CONFLICT.
CloudNativePG omitted six false Boolean attributes from the
reconciled DatabaseRole. PostgreSQL independently confirmed
that identity_owner was NOLOGIN and non-privileged.

PLAT-05B-FIX-2 corrected the parser without relaxing validation.
The exceptional recovery command completed with exit code 0,
preserving attempt_count=2 and lease_version=2.

Ordinary Worker acquisition then completed ServiceDatabases
during attempt 3, with lease_version=3.

The existing Identity DatabaseRole was reused:
eee4ce4c-f650-471a-8e59-01cc13ac0be9.

No tenant, operation, Cluster, PVC or existing role was recreated.
No manual SQL writes, RBAC expansion or retry budget changes occurred.

### Independent acceptance

- Exactly two reconciled DatabaseRoles and two Databases.
- identity database owned by identity_owner.
- clinical database owned by clinical_owner.
- Both roles have LOGIN and no administrative privileges.
- PUBLIC CONNECT revoked on both service databases.
- Independent credentials and matching distribution entries.
- Two successful TLS sessions with correct database and owner.
- Ten negative connection probes rejected by pg_hba or ACL.
- Cross-service access, non-TLS access and connections to
  app, postgres and template1 rejected.
- Three PostgreSQL Clusters healthy and three PVCs Bound.
- Original tenant Cluster UID and persistent volume preserved.
- Argo CD cloudnativepg and saas-platform Synced/Healthy.

### Final durable state

- Tenant: 01a214af-fddc-713e-a908-c1f690486d90.
- Operation: 01a214af-fddc-75c8-9c5e-7f2317a4e6c2.
- Tenant state: PENDIENTE_ADMIN.
- mayAuthenticate=false according to the tenant state model.
- InfrastructureInstance: CONFIRMED.
- ServiceDatabases: CONFIRMED.
- ClinicalPreparation: PENDING.
- IdentityPreparation: PENDING.
- Operation: FAILED_RETRYABLE / ADAPTER_NOT_CONFIGURED.
- manual_intervention_required=true.
- No lease and no scheduled next attempt.
- AttemptCount=3 and LeaseVersion=3.
- Catalog cardinality: one tenant and one operation.
- Worker stopped at 0/0.

### Sanitized evidence

Retained locally outside application and infrastructure repositories:

- identity-role-before-recovery-20261126T074739Z.json
- final-acceptance-20261126T075719Z.json

No credentials, passwords, connection strings or Secret contents
are included in repository evidence.

### Boundaries and follow-up

- Actual Identity and Clinical application consumption remains pending.
- Multi-tenant isolation under multiple live tenants is not certified.
- TLS encryption verified; server certificate identity is not certified.
- Backup/restore, high availability and load testing remain outside scope.
- Exceptional recovery demonstrates reuse after this specific incident,
  not exhaustive recovery from all possible provisioning failures.
- The corrected Worker image is a local laboratory artifact, not
  a production release.
- Platform must remove the temporary exceptional recovery command
  before promotion to main.
- Only one attempt remains under the current four-attempt budget.
  Review this constraint before later provisioning increments.
  No budget adjustment was made during this experiment.
- LAB-10 remains postponed.
