# Plural Migration Prompt Library

This repository helps Argo CD and Flux users migrate their GitOps resources to Plural deployment CRDs.

The repository is organized as a small set of reusable skills plus sample fixtures. The intended workflow is:

1. Pick the skill that matches your source system.
2. Give that skill your current manifests plus context about clusters, namespaces, secrets, and promotion intent.
3. Review the generated Plural CRDs.
4. Validate the generated output before applying it.

## Which skill to use

### Argo CD users

Use `skills/argo-cd-to-plural/` to migrate from:

- `Application`
- `ApplicationSet`
- `AppProject`

It helps map Argo resources into Plural CRDs such as:

- `ServiceDeployment`
- `GlobalService`
- `Project`
- `Pipeline`
- `NamespaceCredentials`

### Flux users

Use `skills/flux-to-plural/` to migrate from:

- `GitRepository`
- `Kustomization`
- `HelmRelease`
- `HelmRepository`
- Flux image automation resources

It helps map Flux resources into Plural CRDs such as:

- `ServiceDeployment`
- `HelmRepository`
- `PrAutomation`

### Validation

After CRDs are generated, use `skills/validate-plural-crds/` to verify:

- YAML syntax
- CRD kinds and API versions
- basic cross-resource references
- compatibility with installed or target Plural CRDs

## What you should provide

To get useful migration output, provide as much of the following as possible:

- your Argo CD or Flux manifests
- repository URLs and relevant paths
- destination clusters and namespaces
- environment layout such as dev, staging, and prod
- notes about secrets, credentials, RBAC, and rollout constraints
- whether a pattern is fleet replication or staged promotion

## Recommended migration flow

1. Start with the skill for your source system.
2. Ask it to inventory your current resources and map them to Plural CRDs.
3. Review assumptions carefully, especially around clusters, secrets, SCM connections, and promotion intent.
4. Compare the generated output with the fixtures in `samples/` if you want a quick sanity check.
5. Run the validation skill before applying anything.

## Example prompts

### For Argo CD

```text
Use $argo-cd-to-plural to convert these Argo CD manifests into Plural deployment CRDs.
Preserve the intent of Applications, decide correctly between GlobalService and Pipeline for ApplicationSets, and call out all assumptions.
```

### For Flux

```text
Use $flux-to-plural to convert these Flux manifests into Plural deployment CRDs.
Translate Kustomizations and HelmReleases into ServiceDeployment resources, handle HelmRepository reuse correctly, and call out all assumptions.
```

### For validation

```text
Use $validate-plural-crds to verify these generated Plural CRDs.
Check YAML validity, resource references, and schema compatibility with the target Plural environment.
```

## Samples

Use `samples/` if you want to see the expected shape of migrations before running the skills on your own manifests.

Included samples cover:

- Argo CD application migration
- Argo CD promotion versus replication
- Flux workload migration
- Flux image automation migration

Current sample layout:

- `samples/argo-application/` — Argo CD Application, ApplicationSet (fleet replication), AppProject
- `samples/argo-promotion/` — Argo CD ApplicationSet as a promotion pipeline with approval gate
- `samples/flux-workload/` — Flux Kustomization and HelmRelease workloads
- `samples/flux-automation/` — Flux image automation mapped to PrAutomation

Note: `samples/argo-promotion/input.md` is a prose description of the intent rather than a YAML manifest, to show that migration skills accept plain-language migration notes alongside or instead of source YAML.

## Reference sources

The migration skills in this repository are meant to align with:

- [docs.plural.sh](https://docs.plural.sh)
- repositories under [github.com/pluralsh](https://github.com/pluralsh) organization, especially [github.com/pluralsh/console](https://github.com/pluralsh/console)
