# gitops-example-iac-argocd-apps
A chart to demonstrate ArgoCD application deployments to many environment using a flat file structure

```sh
helm upgrade -i openshift-gitops-operator setup/helm/openshift-gitops-operator -n openshift-operators
```

```sh
helm upgrade -i bootstrap-rootapp setup/helm/bootstrap-rootapp -n openshift-gitops -f setup/helm/bootstrap-rootapp/values-dev.yaml
```

Openshift GitOps
This argocd instance has been given cluster admin privileges to provision namespaces and bootstrap argocd tenant deployments and tenant rootapps for organizations.

![Openshift GitOps Bootstrap Rootapp](.img/openshift-gitops-bootstrap-rootapp.png)

![Org1 Rootapp](.img/org1-argocd-rootapp.png)