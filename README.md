# Cluster Housekeeping

[![Pipeline](https://github.com/carhartl/cluster-housekeeping/actions/workflows/pipeline.yml/badge.svg)](https://github.com/carhartl/cluster-housekeeping/actions/workflows/pipeline.yml)

## Prerequisites

[minikube](https://minikube.sigs.k8s.io/docs/), [Skaffold](https://skaffold.dev/docs/), [Helm](https://helm.sh/docs/):

```bash
brew install minikube skaffold helm
```

## Development

The implementation was build on Kubernetes 1.26.3. Start a [minikube](https://minikube.sigs.k8s.io/docs/) cluster:

```bash
minikube start --kubernetes-version 1.26.3
```

Then start [Skaffold](https://skaffold.dev/docs/) in continuous watch mode:

```bash
skaffold dev
```

## Observe results

```bash
kubectl logs -l service=housekeeping -n housekeeping -f
```

Deploy a non-compliant workload:

```bash
kubectl apply -f - << EOF
apiVersion: v1
kind: Pod
metadata:
  name: wrong-nginx
spec:
  containers:
    - image: nginx
      name: nginx
EOF
```

Deploy a compliant workload:

```bash
kubectl apply -f - << EOF
apiVersion: v1
kind: Pod
metadata:
  name: good-nginx
  labels:
    team: test
spec:
  containers:
    - image: bitnami/nginx
      name: nginx
EOF
```

Output (evaluated once per minute):

```
{"pod":"good-nginx","rule_evaluation":[{"name":"image_prefix","valid":true},{"name":"team_label_present","valid":true},{"name":"recent_start_time","valid":true}]}
{"pod":"wrong-nginx","rule_evaluation":[{"name":"image_prefix","valid":false},{"name":"team_label_present","valid":false},{"name":"recent_start_time","valid":true}]}
```

## Notes

### Implementation

Assumptions:

- We're not supposed to evaluate pods in the `kube-system` as well as the `housekeeping` namespace, where the service is deployed to.
- We're not supposed to evaluate init containers.
- Only running pods are evaluated.

### Distribution

The implementation is supposed to be distributed as a Helm chart. The chart is available in the `charts` directory, but isn't released (releasable) to a repository yet.
