# argocd-multi-tenancy

This is an example multi-tenant CI/CD implementation using `ApplicationSet`. The [ApplicationSet controller](https://github.com/argoproj-labs/applicationset) is in early stage of development and some features are not yet implemented ([#30](https://github.com/argoproj-labs/applicationset/issues/30), [#114](https://github.com/argoproj-labs/applicationset/issues/114), [#106](https://github.com/argoproj-labs/applicationset/issues/106)...).

## Requirements

* `devs` should manage their application deployments with a `self-service` to ensure the platform teams doesn't burn out
* platform team wants to offer an **opinionated CI/CD pipeline** and it should be **extensible** for the devs
* tenant management is key: **enforce strong isolation between** all teams: dedicated project per team with namespace enforcment `team-a-*` / `team-b-*` and permissions.
* one team can have **multiple apps** in **one or more namespaces**
* one team can only access their own stuff
* gitops for compliance <3
* application code and infra code must be in a separate repository

## Implementation

* `argo-platform-team` (this repo) is the platform team's area of responsibility.
* `argo-team-X-app-Y` is a tenant repository that contains the k8s manifests.


### Process

A tenant has to register a new application in `$ENV/values.yaml`.

```yaml
team-a:
  name: team-a
  project: team-a
  repos:
  - url: https://github.com/moolen/argo-team-a-app-1.git
    path: "env/dev/*"
    ns_prefix: team-a-app1-
  # ...register new app...
  - url: https://github.com/moolen/argo-team-a-app-2.git
    path: "env/dev/*"
    ns_prefix: team-a-app2-
```

Once this is merged the `ApplicationSet` will generate a new `Application` for this repository. The Application will be assigned to `project: team-a` and underlies its namespace and role constraints. A team is not able to deploy into the `argocd` or `kube-system` namespace.
The dev team is free to create new stages e.g. `testing/acceptance/review` without the review of the platform team:

```
argo-team-a-app-1
├── base
└── env
    ├── dev
    │   ├── dev-foo
    │   ├── ...add..new..stage..on..demand
    │   └── dev-integration
    ├── prod
    │   └── prod
    └── staging
        ├── staging-bar
        └── staging-foo
```
