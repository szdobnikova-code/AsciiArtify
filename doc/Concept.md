# AsciiArtify — Kubernetes Local Development Concept

## Introduction

AsciiArtify is a startup developing a Machine Learning based application for converting images into ASCII art.

The team plans to use Kubernetes for application deployment and scaling and needs to choose a Kubernetes-based solution for local development and the initial Proof of Concept (PoC).

Three tools were evaluated:

- **Minikube** — local Kubernetes focused on application development and learning.
- **kind (Kubernetes IN Docker)** — Kubernetes clusters where nodes run as containers, primarily focused on testing and CI.
- **k3d** — a lightweight tool for running k3s clusters inside containers.

All three tools were practically tested in the same GitHub Codespaces environment:

- Linux x86_64
- Docker
- kubectl

The goal is to compare their capabilities, advantages, limitations and container runtime requirements and select the most appropriate solution for the AsciiArtify PoC.

---

## Comparison

| Criteria | Minikube | kind | k3d |
|---|---|---|---|
| **Primary purpose** | Local Kubernetes development and learning | Kubernetes testing and CI | Lightweight local development and PoC |
| **Kubernetes distribution** | Kubernetes | Kubernetes | k3s |
| **Linux** | ✅ | ✅ | ✅ |
| **macOS** | ✅ | ✅ | ✅ |
| **Windows** | ✅ | ✅ | ✅ |
| **amd64 / arm64** | ✅ | ✅ | ✅ |
| **Container-based cluster** | ✅ | ✅ | ✅ |
| **Multi-node cluster** | ✅ | ✅ | ✅ |
| **Automation / CI** | Good | ⭐ Excellent | Good |
| **Deployment speed** | Fast | Fast | ⭐ Very fast |
| **Resource usage** | Moderate | Low | ⭐ Low |
| **Dashboard** | ✅ Built-in add-on | ❌ | ❌ |
| **Additional features** | Add-ons, dashboard, mounts, LoadBalancer support | Multi-node/HA test clusters, custom networking | Load balancer, port mapping, local registry, multi-node |
| **Load Balancer** | Available with additional setup | Requires configuration | ⭐ Created by default |
| **Docker** | ✅ | ✅ | ✅ |
| **Podman** | ⚠️ Experimental driver | ⭐ Supported | ⚠️ Less mature than Docker workflow |
| **Configuration complexity** | Low | Low | ⭐ Low |
| **Documentation / community** | ⭐ Excellent | ⭐ Excellent | Good |
| **Local development** | ⭐ Excellent | Good | ⭐ Excellent |
| **CI / testing** | Good | ⭐ Excellent | Good |
| **PoC suitability** | Good | Good | ⭐ Excellent |
| **Best use case** | Development and Kubernetes learning | CI and Kubernetes testing | **Lightweight application PoC** |

---

## Practical Evaluation

Each tool was installed and tested using the same basic scenario:

1. Create a local Kubernetes cluster.
2. Verify the cluster using `kubectl`.
3. Inspect the Kubernetes node.
4. Inspect the containers created by the tool.
5. Delete the cluster before testing the next solution.

### Minikube

The cluster was created using the Docker driver:

```bash
minikube start --driver=docker
```

Cluster status:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   19s   v1.35.1
```

`minikube status` confirmed:

```text
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

Docker showed one dedicated Minikube container based on the `kicbase` image.

**Practical impression:** Minikube was easy to start and provided a clear local Kubernetes environment. It also offers many developer-oriented features, although some of them are unnecessary for a small PoC.

---

### kind

The cluster was created with:

```bash
kind create cluster --name asciiartify
```

Cluster status:

```text
NAME                        STATUS   ROLES           AGE   VERSION
asciiartify-control-plane   Ready    control-plane   53s   v1.36.1
```

Docker showed one Kubernetes node container:

```text
asciiartify-control-plane
```

based on the `kindest/node` image.

**Practical impression:** kind provides a minimal and predictable Kubernetes environment. Its container-based node model is especially convenient for automated testing and CI.

---

### k3d

The cluster was created with:

```bash
k3d cluster create asciiartify
```

Cluster status:

```text
NAME                       STATUS   ROLES           AGE   VERSION
k3d-asciiartify-server-0   Ready    control-plane   19s   v1.35.5+k3s1
```

Docker showed two containers:

```text
k3d-asciiartify-server-0
k3d-asciiartify-serverlb
```

The first container runs the k3s server and the second provides the cluster load balancer.

**Practical impression:** k3d provided a lightweight cluster with very little configuration while also creating useful networking infrastructure by default.

---

## Advantages and Disadvantages

### Minikube

**Advantages:**

- Easy installation and usage.
- Supports Linux, macOS and Windows.
- Rich set of add-ons.
- Built-in Kubernetes dashboard.
- Multiple drivers and container runtimes.
- Good documentation and large community.
- Suitable for Kubernetes learning and local application development.

**Disadvantages:**

- Can consume more resources than lightweight alternatives.
- Provides more functionality than required for a small PoC.
- Some development workflows rely on Minikube-specific commands.
- Podman driver is currently experimental.

### kind

**Advantages:**

- Lightweight and fast.
- Uses Kubernetes nodes running as containers.
- Uses upstream Kubernetes.
- Excellent for automated tests and CI.
- Supports multi-node clusters.
- Supports Docker, Podman and nerdctl.
- Good reproducibility between environments.

**Disadvantages:**

- Primarily designed for Kubernetes testing.
- No built-in dashboard.
- Fewer developer-oriented features than Minikube.
- Ingress and external service access can require additional configuration.

### k3d

**Advantages:**

- Very fast cluster creation and deletion.
- Low resource requirements.
- Simple CLI.
- Supports multi-node clusters.
- Built-in load balancer.
- Convenient port mapping.
- Local container registry integration.
- Well suited for lightweight development environments and PoCs.

**Disadvantages:**

- Uses k3s rather than upstream Kubernetes directly.
- Fewer built-in developer add-ons than Minikube.
- Podman integration is less mature than the standard Docker workflow.
- Some k3s behavior can differ from other Kubernetes distributions used in production.

---

## Docker Licensing and Podman

The choice of local Kubernetes tool also requires consideration of the container runtime.

Docker Desktop has separate commercial licensing conditions. It is free for:

- personal use;
- education;
- non-commercial open-source projects;
- small businesses with fewer than 250 employees **and** less than $10 million USD in annual revenue.

Larger organizations require a paid Docker subscription for professional Docker Desktop usage.

For the current AsciiArtify startup stage this may not be a limitation, but Docker Desktop licensing can become a consideration as the company grows.

An alternative is **Podman**, an open-source container engine that can be used without Docker Desktop.

Podman support differs between the evaluated tools:

| Tool | Podman support |
|---|---|
| **Minikube** | Podman driver exists but is currently marked experimental |
| **kind** | Docker, Podman and nerdctl can be used as providers |
| **k3d** | Primarily designed around a Docker-compatible runtime; Podman integration is less straightforward |

If avoiding Docker Desktop becomes a strict requirement, **kind + Podman** would be the strongest alternative among the evaluated solutions.

For the initial AsciiArtify PoC, this does not outweigh the development advantages provided by k3d.

---

## Demo — Hello World with k3d

Based on the comparison and practical evaluation, **k3d was selected for the AsciiArtify PoC**.

To validate the proposed local Kubernetes environment, a simple `nginx` application was deployed and accessed through a Kubernetes Service.

The recorded demo includes:

1. Creating the `asciiartify` k3d cluster.
2. Verifying that the k3s control-plane node is in the `Ready` state.
3. Creating the `hello-world` deployment based on the `nginx` image.
4. Verifying that the deployment is available and the pod is running.
5. Exposing the deployment using a `ClusterIP` Kubernetes Service.
6. Forwarding the service port to `localhost:8080`.
7. Sending an HTTP request to the deployed application.
8. Receiving the **Welcome to nginx!** response.
9. Verifying the final state of the pod and service.

### Demo result

The deployment was successfully created:

```text
NAME          READY   UP-TO-DATE   AVAILABLE
hello-world   1/1     1            1
```

The application pod reached the `Running` state:

```text
NAME                           READY   STATUS    RESTARTS
hello-world-86df7f6cdb-vggw8   1/1     Running   0
```

The application was exposed through a Kubernetes Service:

```text
NAME          TYPE        PORT(S)
hello-world   ClusterIP   80/TCP
```

The service was forwarded to the local environment:

```bash
kubectl port-forward service/hello-world 8080:80
```

and verified with:

```bash
curl http://localhost:8080
```

The HTTP response contained:

```html
<h1>Welcome to nginx!</h1>
```

This confirms that the complete Kubernetes flow — **cluster → deployment → pod → service → application response** — works successfully in the selected k3d environment.

### Recorded Demo

[![asciicast](https://asciinema.org/a/EV8Fy1QBstTPLSnU.svg)](https://asciinema.org/a/EV8Fy1QBstTPLSnU)

_Click the terminal preview to open the recorded demo._

---

## Conclusion

All three evaluated tools successfully created working Kubernetes clusters during practical testing, but each solution is optimized for a different use case.

### Minikube

Recommended when the team needs a feature-rich local Kubernetes environment with a dashboard, add-ons and tools that make Kubernetes easier to learn and explore.

### kind

Recommended when the primary goal is Kubernetes testing, CI automation or stronger Podman compatibility.

### k3d

**Recommended for the AsciiArtify PoC.**

k3d provides the best balance for the current project because it offers:

- fast cluster creation;
- low resource consumption;
- simple configuration;
- convenient networking;
- a load balancer by default;
- multi-node cluster support;
- an easy path from a minimal local environment to a more realistic PoC topology.

The practical test also demonstrated that an application can be deployed and exposed with very little additional configuration.

### Final Recommendation

| Scenario | Recommended Tool |
|---|---|
| Kubernetes learning / feature-rich local development | **Minikube** |
| CI / automated Kubernetes testing | **kind** |
| Podman-first environment | **kind** |
| Lightweight local PoC | **k3d** |
| **AsciiArtify PoC** | **k3d** ⭐ |

> **Decision: use k3d as the local Kubernetes environment for the initial AsciiArtify PoC.**