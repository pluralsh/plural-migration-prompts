---
name: argo-cd-to-plural
description: Convert Argo CD applications, application sets, and projects into Plural deployment CRDs. Use when the task is to migrate Argo CD resources to Plural and decide between ServiceDeployment, GlobalService, Project, NamespaceCredentials, and Pipeline.
---

# Argo CD To Plural

Use this skill when the source system is Argo CD and the target is Plural deployment CRDs.

## When to use

Use this skill when the source includes:

- `Application`
- `ApplicationSet`
- `AppProject`

This skill is most useful when the main migration decision is whether a source object should become:

- `ServiceDeployment`
- `GlobalService`
- `Pipeline`

## Recommended targets

Start with these recommended Plural CRDs when designing migrations:

- `GitRepository`
- `Cluster`
- `ServiceDeployment`
- `GlobalService`
- `Project`
- `ServiceContext`
- `NamespaceCredentials`
- `Pipeline`
- `PrAutomation`
- `HelmRepository`

## Core rules

- Use the `deployments.plural.sh` API group and choose the resource version that matches the target Plural environment.
- Reuse shared Git sources instead of redefining them per workload.
- Prefer existing clusters over creating new `Cluster` resources unless the input explicitly asks for cluster registration. When a cluster name is present in the input but no `Cluster` CRD is generated, record it as an assumption.
- Reference clusters using `spec.cluster` (the cluster handle name) on `ServiceDeployment` and `GlobalService`. Only use `clusterRef` when the input explicitly provides a full Cluster CRD object reference.
- Do not invent cluster ids, secret names, SCM connection names, project ids, or RBAC bindings.
- Put missing operational data into assumptions and review notes.
- Keep the output idiomatic to Plural rather than mirroring every source-system detail.

## Resource scopes

Know whether each resource is cluster-scoped or namespace-scoped before generating manifests:

| Resource | Scope |
|---|---|
| `GitRepository` | Cluster-scoped — omit `metadata.namespace` |
| `Project` | Cluster-scoped — omit `metadata.namespace` |
| `PrAutomation` | Cluster-scoped — omit `metadata.namespace` |
| `NamespaceCredentials` | Cluster-scoped — omit `metadata.namespace` |
| `ServiceDeployment` | Namespace-scoped — always set `metadata.namespace` |
| `GlobalService` | Namespace-scoped — always set `metadata.namespace` |
| `Pipeline` | Namespace-scoped — always set `metadata.namespace` |
| `HelmRepository` | Namespace-scoped — always set `metadata.namespace` |
| `Cluster` | Namespace-scoped — always set `metadata.namespace` |
| `ServiceContext` | Namespace-scoped — always set `metadata.namespace` |

Cross-resource references (`repositoryRef`, etc.) to cluster-scoped resources must omit `namespace` in the reference object. For cluster targeting, use `spec.cluster` with the cluster handle name instead of a `clusterRef` object.

## Argo translation rules

1. Translate each `Application` into one `ServiceDeployment` unless the input clearly describes a promotion pipeline instead of a standalone workload.
2. Map `spec.source.repoURL` to a reusable `GitRepository` when the source is Git.
3. Map `spec.source.path` to `spec.git.folder`.
4. Map `spec.source.targetRevision` to `spec.git.ref` for Git sources or `spec.helm.version` for external Helm charts.
5. Map `spec.source.chart` to `spec.helm.chart`, `spec.source.helm.valueFiles` to `spec.helm.valuesFiles`, and the Helm repository URL to `spec.helm.url`. Only generate a `HelmRepository` CRD when authentication credentials are required; for public repositories use `helm.url` directly.
6. Map `spec.destination.namespace` to `ServiceDeployment.spec.namespace`.
7. Reference destination clusters with `spec.cluster` (the cluster handle name); do not invent cluster registration unless the input explicitly asks for it.
8. Translate `ApplicationSet` cluster label selectors to `GlobalService.spec.tags` when the intent is fleet replication.
9. Translate `ApplicationSet` to `Pipeline` only when the intent is stage promotion, ordered rollout, or manual approval across environments.
10. Translate `AppProject` to `Project` when the project boundary still matters for tenancy or RBAC; if translated, always add a review note explaining why the boundary was kept.
11. Do not invent direct Plural fields for Argo sync windows, self-heal, or prune policies when there is no exact target field; carry them into review notes.

## Output shape

Every migration response should produce these sections in order:

1. `Inventory`
2. `Mapping`
3. `Assumptions`
4. `Generated artifacts`
5. `Review notes`

## Main prompt

```text
Convert the following Argo CD estate to Plural deployment CRDs.

Prefer these Plural targets unless the target environment or migration scenario clearly requires additional resources:
- GitRepository
- Cluster
- ServiceDeployment
- GlobalService
- Project
- ServiceContext
- NamespaceCredentials
- Pipeline
- PrAutomation
- HelmRepository

Apply the Argo migration rules from this skill.

Return these sections:
1. Inventory table
2. Mapping table
3. Assumptions table
4. Generated artifacts
5. Review notes

Input:
{{ ARGO_MANIFESTS_AND_NOTES }}
```

## Decision prompt

Use this when the source contains `ApplicationSet` and the rollout intent is unclear.

```text
Determine whether this Argo ApplicationSet is acting as:
- fleet replication of the same service to many clusters
- staged promotion across environments
- simple generation convenience that should become separate ServiceDeployment objects

Use these rules:
- Choose GlobalService only for one template applied to many clusters selected by tags or distro.
- Choose Pipeline only for promotion from one stage to the next.
- Choose multiple ServiceDeployment objects when there is no true shared fleet template or stage flow.

Return:
1. chosen Plural target
2. one-paragraph reasoning
3. what additional input would change the decision
```

## Review prompt

```text
Review this generated Argo-to-Plural migration.

Check for:
- duplicated GitRepository definitions for the same repo URL
- ApplicationSets that should have become GlobalService but were emitted as many unrelated services
- promotion flows that should have become Pipeline but were flattened into unrelated services
- invented cluster ids, SCM connections, secrets, or unsupported fields
- missing review notes for Argo behaviors that do not map one-to-one

Return:
1. blocking issues
2. important but non-blocking issues
3. exact artifact corrections
```

## Fixtures

Use these only when you need examples or regression checks:

- `../../samples/argo-application/input.yaml`
- `../../samples/argo-application/output.yaml`
- `../../samples/argo-promotion/input.md`
- `../../samples/argo-promotion/output.yaml`

## Do not

- Do not generate Terraform.
- Do not invent missing operational identifiers.
- Do not treat fleet replication and promotion as the same thing.
- Do not treat this skill as a validation step; use `../validate-plural-crds/SKILL.md` after generation.
