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

PREFLIGHT_PASS.
Integration results pending.
LAB-10 remains postponed.
