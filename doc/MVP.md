# AsciiArtify MVP

## Overview

The MVP stage demonstrates the deployment of the `go-demo-app` application to a Kubernetes cluster using Argo CD and GitOps principles.

The original application repository was forked and used as the Git source for the Argo CD Application:

- Original repository: `https://github.com/den-vasyliev/go-demo-app`
- Fork: `https://github.com/szdobnikova-code/go-demo-app`
- Kubernetes cluster: k3d
- GitOps tool: Argo CD
- Namespace: `demo`

Argo CD tracks the `main` branch of the fork and deploys the Helm chart located in the `helm` directory.

## GitOps Flow

The deployment follows the following flow:

```text
GitHub repository
       |
       v
    Argo CD
       |
       v
 Kubernetes cluster
       |
       v
  go-demo-app
```

Application changes are committed and pushed to GitHub. Argo CD detects changes in the repository and automatically synchronizes the desired state with the Kubernetes cluster.

## Argo CD Application

The application was configured with the following parameters:

| Parameter | Value |
|---|---|
| Application | `go-demo-app` |
| Project | `default` |
| Repository | `https://github.com/szdobnikova-code/go-demo-app.git` |
| Revision | `HEAD` |
| Path | `helm` |
| Cluster | `https://kubernetes.default.svc` |
| Namespace | `demo` |
| Sync policy | Automatic |

Automatic synchronization is enabled, allowing Argo CD to apply changes from the Git repository without manual synchronization.

## Deployment

After synchronization, Argo CD deploys all application components to the `demo` namespace.

The deployment can be verified with:

```bash
kubectl get pods -n demo
```

The deployed application includes the API, frontend, ASCII, image, data, NATS, database, cache, and Ambassador API gateway components.

Application functionality can be verified through the Ambassador gateway:

```bash
kubectl port-forward svc/ambassador -n demo 8081:80
```

and:

```bash
curl http://localhost:8081/api/
```

Example response:

```text
k8sdiy-api:599e1af
```

## Demo 1 — Application Functionality

The first demo shows the application deployed using the configured infrastructure.

The demo verifies:

- the Argo CD Application is `Healthy` and `Synced`;
- application workloads are running in the Kubernetes cluster;
- the deployed application responds through the Ambassador API gateway.

[Watch the application functionality demo](media/mvp-application-demo.mov)

## Demo 2 — Automatic Synchronization

The second demo demonstrates the complete GitOps synchronization cycle.

The number of API replicas in `helm/values.yaml` is changed from:

```yaml
api:
  replicas: 2
```

to:

```yaml
api:
  replicas: 1
```

The change is committed and pushed to the forked GitHub repository.

No manual synchronization is performed in Argo CD.

Argo CD automatically:

1. detects the new Git revision;
2. identifies the application as `OutOfSync`;
3. starts synchronization;
4. applies the new desired state to Kubernetes;
5. returns the application to `Healthy` and `Synced`.

The result is verified in Kubernetes:

```bash
kubectl get pods -n demo -l app=go-demo-app-api
```

and:

```bash
kubectl get deployment go-demo-app-api -n demo
```

After automatic synchronization, the API deployment contains one replica as defined in Git.

[Watch the application functionality demo](media/mvp-auto-sync-demo.mov)

## Result

The MVP demonstrates a complete GitOps deployment workflow:

```text
Code change
    |
    v
Git commit and push
    |
    v
GitHub repository
    |
    v
Argo CD detects the change
    |
    v
Automatic synchronization
    |
    v
Kubernetes cluster is updated
```

The application is successfully deployed to Kubernetes using Argo CD, and changes committed to the Git repository are automatically synchronized with the cluster.
