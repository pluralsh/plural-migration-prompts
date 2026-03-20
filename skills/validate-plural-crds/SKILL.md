---
name: validate-plural-crds
description: Validate generated Plural deployment CRD manifests. Use when CRDs have already been generated and the task is to check YAML validity, schema compatibility, API versions, and cross-resource references before apply or review.
---

# Validate Plural CRDs

Use this skill only after migration output already exists.

Its purpose is to verify that generated Plural CRDs are structurally sound before review or apply.

## When to use

Use this skill when the user wants to verify:

- YAML syntax
- `apiVersion` and `kind` correctness
- compatibility with installed or target CRDs
- simple cross-resource references

## Validation workflow

1. Parse the generated YAML first.
2. Check that only expected Plural CRD kinds are present.
3. Check simple cross-resource references.
4. Prefer server-side validation against a cluster with the target Plural CRDs installed.
5. If server-side validation is not available, compare fields against the target CRD version and public schema sources.

## Fast checks

### YAML validity

Parse the output as YAML before doing anything else.

Examples:

```sh
# Python (commonly available)
python3 -c "import sys, yaml; list(yaml.safe_load_all(open('generated.yaml'))); print('YAML OK')"

# Ruby (if available)
ruby -e 'require "yaml"; YAML.load_stream(File.read("generated.yaml")); puts "YAML OK"'

# kubectl client-side dry run (no cluster needed)
kubectl apply --dry-run=client -f generated.yaml
```

### Resource inventory

Check that the file contains only the expected Plural resources.

Typical targets in this library:

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

### Resource scope check

Verify that cluster-scoped resources have no `metadata.namespace` and that namespace-scoped resources always have `metadata.namespace` set:

| Cluster-scoped (no `metadata.namespace`) | Namespace-scoped (must have `metadata.namespace`) |
|---|---|
| `GitRepository` | `ServiceDeployment` |
| `Project` | `GlobalService` |
| `PrAutomation` | `Pipeline` |
| `NamespaceCredentials` | `HelmRepository` |
| | `Cluster` |
| | `ServiceContext` |

Also verify that cross-resource references to cluster-scoped resources omit `namespace` in the reference object (e.g., `repositoryRef` pointing to `GitRepository` must not include `namespace`).

### Reference sanity

Check simple cross-resource references such as:

- `ServiceDeployment.spec.repositoryRef`
- `ServiceDeployment.spec.clusterRef`
- `GlobalService.spec.serviceRef` or `GlobalService.spec.template`
- `Pipeline` stage service references
- `PrAutomation.spec.repositoryRef` and `PrAutomation.spec.scmConnectionRef`

## Schema checks

The best schema check is against the actual CRDs installed in the target environment.

Preferred options:

1. Use server-side validation against a cluster that already has the Plural CRDs installed.
2. Validate against CRD definitions exported from the target environment.
3. Compare generated fields against the current CRD definitions in [github.com/pluralsh/console](https://github.com/pluralsh/console) if cluster access is not available.

### Server-side dry run

If the target cluster has the Plural CRDs installed:

```sh
kubectl apply --dry-run=server -f generated.yaml
```

This is the strongest practical check because it validates against the actual API server schema.

### Client-side review

If server-side validation is not available, manually confirm:

- the `deployments.plural.sh` API group is correct for the target environment
- the resource version matches the target environment
- fields used in generated manifests exist on the target CRD version

## Common failure patterns

Reject or fix the output if you find:

- invalid YAML
- non-Plural resources mixed into the generated output
- missing required references such as `clusterRef` or `repositoryRef`
- fields that do not exist on the target CRD version
- `GlobalService` used where a `Pipeline` is clearly required
- `Pipeline` used where the source intent is simple fleet replication
- `GlobalService` with `serviceRef` pointing to a `ServiceDeployment` that has an invented `clusterRef`; prefer `template` when no concrete seed cluster exists
- invented secret names, SCM connection names, or cluster identifiers presented as facts
- `metadata.namespace` set on cluster-scoped resources (`GitRepository`, `Project`, `PrAutomation`, `NamespaceCredentials`)
- `namespace` included in cross-resource references to cluster-scoped resources

## Suggested workflow

1. Generate CRDs with `../argo-cd-to-plural/SKILL.md` or `../flux-to-plural/SKILL.md`.
2. Parse the YAML.
3. Check resource kinds and cross-resource references.
4. Run server-side dry run if the target cluster has Plural CRDs installed.
5. Only then move on to human review and rollout planning.

## Do not

- Do not redesign the migration.
- Do not silently assume CRD versions.
- Do not treat generation and validation as the same task.
