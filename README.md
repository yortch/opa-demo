# opa-demo
Demo to deploy using OPA agent into Openshift using helm, argoCD, tekton.
This demo uses OpenShift 4.9 running on AWS (ROSA).

## Pre-requisites (for Phases 2 and 3)
Install the following operators via OperatorHub in OpenShift Console using latest version available
and using default settings:
1. `Red Hat OpenShift GitOps` (version 1.7.0)
1. `Red Hat OpenShift Pipelines` (version 1.7.3)

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
1. Checkout `helm` branch

    ```bash
    git checkout helm
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

1. Checkout `gitops` branch

    ```bash
    git checkout gitops
    ```

1. Change to `iac/gitops` directory:

    ```bash
    cd iac/gitops
    ```

1. Install ArgoCD instance (prerequisite):

    ```bash
    oc apply -f prereqs/argocd.yaml
    ```

1. Install `gitops-app-project` chart:

    ```bash
    helm upgrade -i opa-gitops gitops-app-project -f ../helm/values/dev/values.yaml \
    --set "app.namespace=$NAMESPACE"
    ```

## Phase 3: Deploy using GitOps (ArgoCD) and Tekton pipeline

1. Login to OpenShift console via CLI using `oc login`
1. Export project name to use:

    ```bash
    NAMESPACE=opa-tekton
    ```

1. Create project

    ```bash
    oc new-project $NAMESPACE
    ```
1. Checkout `main` branch (if not already there)

    ```bash
    git checkout main
    ```

1. Change to `iac/gitops` directory:

    ```bash
    cd iac/gitops
    ```

1. Install ArgoCD instance (prerequisite):

    ```bash
    oc apply -f prereqs/argocd.yaml
    ```

1. Install `gitops-app-project` chart:

    ```bash
    helm upgrade -i opa-tekton gitops-app-project -f ../helm/values/dev/values.yaml \
    --set "app.namespace=$NAMESPACE"
    ```

1. This will install OPA Agent but pods will fail to start because of missing configmap.
   Next run the `configure-opa` pipeline from OpenShift console.

### Policy Validation

1. Get route to service either from OpenShift Console or running this command:

     ```bash
     URL=http://$(oc get route $NAMESPACE-opa-chart -o jsonpath='{.spec.host}')
     ```

1. Next attempt to query policies without authentication (expect success `200`):

    ```bash
    curl --location -i $URL/v1/policies/
    ```

1. Next attempt to insert new policy without authentication (expect error `403`):

    ```bash
    curl -i --location --request PUT $URL/v1/policies/example1 \
    --data-raw 'package example1
    default allow := false'
    ```

1. Next obtain a valid token, e.g. from http://jwt.io entering secret 
   (in `Verify Signature`) section and specifying `"client": "test"` in 
   payload and save it on an environment variable named `JWT` (expect success `200`)

    ```bash
    curl --location -i --request PUT $URL/v1/policies/example1 \
    --header "Authorization: Bearer $JWT" \
    --data-raw 'package example1
    default allow := false'
    ```
