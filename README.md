# opa-demo
Demo to deploy using OPA agent into Openshift using helm, argoCD, tekton

## Phase 0: Deploy Kubernetes Resources (POC)

This is the minimum viable configuration. Recommended only for POC purposes.

1. Login to OpenShift console via CLI using `oc login`
1. Export project name to use:

    ```bash
    NAMESPACE=opa
    ```

1. Create project

    ```bash
    oc new-project $NAMESPACE
    ```

1. Create OPA resources:

    ```bash
    oc apply -f iac/resources/deployment.yaml
    oc apply -f iac/resources/service.yaml
    oc apply -f iac/resources/route.yaml
    ```
## Phase 1: Deploy using Helm

1. Login to OpenShift console via CLI using `oc login`
1. Export project name to use:

    ```bash
    NAMESPACE=opa-helm
    ```

1. Create project

    ```bash
    oc new-project $NAMESPACE
    ```

1. Change to `iac/helm` directory:

    ```bash
    cd iac/helm
    ```

1. Install `helm` chart:

    ```bash
    helm upgrade -i opa-helm opa-chart -f values/dev/values.yaml
    ```
## Phase 2: Deploy using GitOps (ArgoCD)

1. Login to OpenShift console via CLI using `oc login`
1. Export project name to use:

    ```bash
    NAMESPACE=opa-gitops
    ```

1. Create project

    ```bash
    oc new-project $NAMESPACE
    ```

1. Change to `iac/gitops` directory:

    ```bash
    cd iac/gitops
    ```

1. Install `gitops-app-project` chart:

    ```bash
    helm upgrade -i opa-gitops gitops-app-project -f ../helm/values/dev/values.yaml --set "app.namespace=$NAMESPACE"
    ```