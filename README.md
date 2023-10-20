# gitops-example-iac-argocd-apps

A chart to demonstrate ArgoCD application deployments from tenant ArgoCD instances, bootstrapped by a cluster-admin ArgoCD instance using a flat file structure/monorepo.

## Setup

A cluster admin simply needs to deploy the Openshift GitOps operator...

```sh
helm upgrade -i openshift-gitops-operator setup/helm/openshift-gitops-operator -n openshift-operators
```

and then bootstrap the rootapp correlating to the intended SDLC environment...

```sh
helm upgrade -i bootstrap-rootapp setup/helm/bootstrap-rootapp -n openshift-gitops -f setup/helm/bootstrap-rootapp/values-dev.yaml
```

## Explanation

### Openshift GitOps ArgoCD Bootstrap RootApp:

The Openshift GitOps argocd instance service account has been given cluster admin privileges to provision namespaces and bootstrap tenant argocd rootapp deployments per organization.
A cluster admin needs to maintain the [./setup/helm/bootstrap-rootapp/values.yaml](./setup/helm/bootstrap-rootapp/values.yaml) yaml file with a list of orgs and repo urls for each org's monorepo. Each repoURL should have a `helm/rootapp` and `helm/argocd` chart to instantiate the tenant argocd and applications.

![Openshift GitOps Bootstrap Rootapp](.img/openshift-gitops-bootstrap-rootapp.png)

### Org1 ArgoCD RootApp:

This is an argocd instance to manage many applications for many teams within the Org1 organization. It was bootstrapped from the Openshift GitOps instance.

![Org1 Rootapp](.img/org1-argocd-rootapp.png)

### Org2 ArgoCD RootApp

This is an argocd instance to manage many applications for many teams within the Org1 organization. It was bootstrapped from the Openshift GitOps instance.

![Org2 Rootapp](.img/org2-argocd-rootapp.png)

### Bootstrapping

The code within the [./setup/helm](./setup/helm) directory is intended to be ran by a cluster admin once per SDLC cluster.

### Org MonoRepo

The org monorepo needs to contain its own `helm/argocd` and `helm/rootapp` helm charts since the bootstrapping from Openshift Gitops expects these paths. The rootapp can then instantiate the `helm/applications` chart to deploy and tenant apps. Notice that there is also a `helm/appproject` used when dynamically generating AppProjects based on a folder structure.

```sh
[tbox@fedora gitops-example-iac-argocd-apps]$ tree helm/
helm/
├── applications
│   ├── Chart.yaml
│   ├── templates
│   │   ├── applicationset-team-apps.yaml
│   │   ├── applicationset-team-projects.yaml
│   │   └── _helpers.tpl
│   └── values.yaml
├── appproject
│   ├── Chart.yaml
│   ├── templates
│   │   ├── appproject.yaml
│   │   └── _helpers.tpl
│   └── values.yaml
├── argocd
│   ├── Chart.yaml
│   ├── templates
│   │   ├── argocd.yaml
│   │   └── _helpers.tpl
│   └── values.yaml
└── rootapp
    ├── Chart.yaml
    ├── templates
    │   ├── application-rootapp.yaml
    │   └── _helpers.tpl
    └── values.yaml
```

The ApplicationSet [./helm/applications/templates/applicationset-team-projects.yaml](./helm/applications/templates/applicationset-team-projects.yaml) uses a [Git Generator Directories](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Git/#git-generator-directories) subtype to dynamically create AppProjects based on a directory.

```yaml
...
  generators:
    - git:
        repoURL: 'https://github.com/trevorbox/gitops-example-iac-argocd-apps.git'
        revision: HEAD
        directories:
          - path: {{ printf "clusters/%s/orgs/%s/teams/*" .Values.cluster.name .Values.org.name }}
...
```

The ApplicationSet [./helm/applications/templates/applicationset-team-apps.yaml](./helm/applications/templates/applicationset-team-apps.yaml) uses a [Git Generator Files](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Git/#git-generator-files) subtype to dynamically create Applications based yaml files within an expected directory.

```yaml
...
  generators:
    - git:
        repoURL: 'https://github.com/trevorbox/gitops-example-iac-argocd-apps.git'
        revision: HEAD
        files:
          - path: {{ printf "clusters/%s/orgs/%s/teams/*/contexts/*/apps/*.yaml" .Values.cluster.name .Values.org.name }}
...
```

Each ApplicationSet references a path structure where teams would be responsible for submitting pull requests to for changing their application's deployment code reference.

```sh
[tbox@fedora gitops-example-iac-argocd-apps]$ tree clusters
clusters
├── dev
│   └── orgs
│       ├── org1
│       │   └── teams
│       │       ├── team1
│       │       │   └── contexts
│       │       │       ├── context1
│       │       │       │   └── apps
│       │       │       │       └── go-app.yaml
│       │       │       └── context2
│       │       │           └── apps
│       │       │               └── go-app.yaml
│       │       └── team2
│       │           └── contexts
│       │               └── context1
│       │                   └── apps
│       │                       ├── go-app2.yaml
│       │                       └── go-app.yaml
│       └── org2
│           └── teams
│               ├── team1
│               │   └── contexts
│               │       ├── context1
│               │       │   └── apps
│               │       │       └── go-app.yaml
│               │       └── context2
│               │           └── apps
│               │               └── go-app.yaml
│               └── team2
│                   └── contexts
│                       └── context1
│                           └── apps
│                               ├── go-app2.yaml
│                               └── go-app.yaml
...
```