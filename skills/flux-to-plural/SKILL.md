---
name: flux-to-plural
description: Convert Flux GitOps resources into Plural deployment CRDs. Use when the task is to migrate Flux resources such as GitRepository, Kustomization, HelmRelease, HelmRepository, or image automation into Plural CRDs.
---

# Flux To Plural

Use this skill when the source system is Flux and the target is Plural deployment CRDs.

## When to use

Use this skill when the source includes:

- `GitRepository`
- `Kustomization`
- `HelmRelease`
- `HelmRepository`
- `ImageRepository`
- `ImagePolicy`
- `ImageUpdateAutomation`

This skill is most useful when the main migration decision is whether a source object should become:

- `ServiceDeployment`
- `HelmRepository`
- `PrAutomation`

## Recommended targets

Start with these recommended Plural CRDs when designing migrations:

- `GitRepository`
- `Cluster`
- `ServiceDeployment`
- `GlobalService` — use when Flux `Kustomization` or `HelmRelease` is replicated across many clusters via label selectors or fleet tooling
- `Project` — use when the source has explicit tenant or team isolation that must be preserved
- `ServiceContext` — use when shared configuration must be injected into multiple services
- `NamespaceCredentials` — use when per-namespace operator credentials are required for multi-tenant deployments
- `Pipeline` — use when Flux sources describe a promotion sequence across environments
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

## Flux translation rules

1. Deduplicate each Flux `GitRepository` into one Plural `GitRepository`.
2. Translate each `Kustomization` into one `ServiceDeployment` with `repositoryRef` plus `git.folder`.
3. Strip a leading `./` from Flux paths when translating into `git.folder`.
4. Translate Flux `dependsOn` into `ServiceDeployment.dependencies` only when the dependency is a real runtime prerequisite.
5. Translate each `HelmRelease` into one `ServiceDeployment` using `spec.helm`.
6. Map `HelmRelease.spec.chart.spec.chart` to `helm.chart`, `version` to `helm.version`, and the Helm repository URL to `helm.url`. Only generate a `HelmRepository` CRD when authentication credentials are required for the chart repository; for public repositories always use `helm.url` directly on the `ServiceDeployment`.
7. Map inline Flux Helm values to `helm.values` and values sourced from secrets or config maps to `helm.valuesFrom` or `helm.valuesConfigMapRef` when supported by the target CRD version.
8. Translate `targetNamespace` to `ServiceDeployment.spec.namespace`.
9. Carry unsupported Flux behavior such as `suspend` or fine-grained interval tuning into review notes when there is no exact Plural field required.
10. Translate Flux image automation to `PrAutomation` only when repo-writing automation is still desired after migration; otherwise keep it as a manual follow-up item.

## Output shape

Every migration response should produce these sections in order:

1. `Inventory`
2. `Mapping`
3. `Assumptions`
4. `Generated artifacts`
5. `Review notes`

## Main prompt

```text
Convert the following Flux estate to Plural deployment CRDs.

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

Apply the Flux migration rules from this skill.

Return these sections:
1. Inventory table
2. Mapping table
3. Assumptions table
4. Generated artifacts
5. Review notes

Input:
{{ FLUX_MANIFESTS_AND_NOTES }}
```

## Review prompt

```text
Review this generated Flux-to-Plural migration.

Check for:
- duplicated GitRepository definitions for the same Git URL
- HelmRelease output that loses chart source information or forces an unnecessary non-CRD framing
- Kustomization output that drops dependsOn or targetNamespace without explanation
- image automation forced into ServiceDeployment instead of PrAutomation or review notes
- missing notes for suspend, interval, or other behavior without an exact Plural mapping

Return:
1. blocking issues
2. important but non-blocking issues
3. exact artifact corrections
```

## Fixtures

Use these only when you need examples or regression checks:

- `../../samples/flux-workload/input.yaml`
- `../../samples/flux-workload/output.yaml`
- `../../samples/flux-automation/input.yaml`
- `../../samples/flux-automation/output.yaml`

## Do not

- Do not generate Terraform.
- Do not force image automation into `ServiceDeployment`.
- Do not invent missing operational identifiers.
- Do not treat this skill as a validation step; use `../validate-plural-crds/SKILL.md` after generation.
