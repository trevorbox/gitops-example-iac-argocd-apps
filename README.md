# gitops-example-iac-argocd-apps
A chart to demonstrate ArgoCD application deployments to many environment using a flat file structure

```sh
helm upgrade -i openshift-gitops-operator setup/helm/openshift-gitops-operator -n openshift-operators
```

```sh
helm upgrade -i bootstrap-rootapp setup/helm/bootstrap-rootapp -n openshift-gitops -f setup/helm/bootstrap-rootapp/values-dev.yaml
```