# Argo Workflows

https://argoproj.github.io/argo-workflows/

---

## Installation

### CLI installation

```bash
curl -sLO https://github.com/argoproj/argo/releases/download/v3.0.0-rc2/argo-linux-amd64.gz
gunzip argo-linux-amd64.gz
chmod +x argo-linux-amd64
mv ./argo-linux-amd64 /usr/local/bin/argo
argo version
```

### k3d cluster

```bash
k3d cluster create argo
kubectl create rolebinding default-admin --clusterrole=admin --serviceaccount=default:default
```

### Argo Workflows

```bash
kubectl create ns argo
kustomize build ./argo/overlays/k3d  | kubectl apply -n argo -f -
```

### Argo UI

```bash
kubectl -n argo port-forward deployment/argo-server 2746:2746
```

### Minio Installation

https://github.com/argoproj/argo-workflows/blob/master/docs/configure-artifact-repository.md

```bash
helm repo add minio https://helm.min.io/
helm repo update
helm install argo-artifacts minio/minio --set service.type=LoadBalancer --set fullnameOverride=argo-artifacts

Minio can be accessed via port 9000 on an external IP address. Get the service external IP address by:
kubectl get svc --namespace default -l app=argo-artifacts

Note that the public IP may take a couple of minutes to be available.

You can now access Minio server on http://<External-IP>:9000. Follow the below steps to connect to Minio server with mc client:

  1. Download the Minio mc client - https://docs.minio.io/docs/minio-client-quickstart-guide

  2. Get the ACCESS_KEY=$(kubectl get secret argo-artifacts --namespace default -o jsonpath="{.data.accesskey}" | base64 --decode) and the SECRET_KEY=$(kubectl get secret argo-artifacts --namespace default -o jsonpath="{.data.secretkey}" | base64 --decode)
  3. mc alias set argo-artifacts http://<External-IP>:9000 "$ACCESS_KEY" "$SECRET_KEY" --api s3v4

  4. mc ls argo-artifacts

Alternately, you can use your browser or the Minio SDK to access the server - https://docs.minio.io/categories/17

```

---

## Running Argo

### Submit a workflow

```bash
argo submit workflows/coin-flip.yaml
```

### Watch latest workflow

```bash
argo watch @latest
```

### Creating a workflow template and using template in a workflow

The template needs to be registered in kubernetes.

```bash
argo template create workflow-templates/workflow-template-inner-steps.yaml
```

It can then be used in a workflow and the workflow submitted.

```bash
argo submit workflows/inner-steps-from-template.yaml
```
