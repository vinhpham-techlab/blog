---
title: Kubernetes Part 1 - Deploying a local cluster
subtitle: Using KinD (Kubernetes in Docker)
date: 2025-10-15
tags: [kubernetes, practice, kind, calico, openebs]
---

KinD (Kubernetes in Docker) enables developers to deploy lightweight local Kubernetes clusters by running nodes as Docker containers, minimizing resource use compared to virtual machines. With Docker and KinD installed, users can create multi-node clusters quickly using a single command or config file, ideal for testing Ingress, RBAC, or service meshes on resource-constrained machines like laptops. Tools like MetalLB enhance testing capabilities, making KinD a fast, portable, and cost-effective solution for Kubernetes development and prototyping.

## Prerequisites

Before starting, ensure the following are installed on your system:

- **Docker**: Required to run KinD clusters.
- **kubectl**: For interacting with the Kubernetes cluster.
- **kustomize**: To apply Helm-based manifests for Calico and OpenEBS.
- A compatible OS (Linux, macOS, or Windows with Docker support).
- *[Optional]*: Clone the repository for local access to configuration files:
  
```sh
git clone https://gitlab.com/vinhpham-techlab/infrastructure.git
cd infrastructure/clusters/vinhphamtechlab
```

## Step 1: Install KinD

KinD runs Kubernetes nodes as Docker containers, reducing resource overhead compared to virtual machines. To install KinD on an M1/ARM Mac, run the following commands:

```sh
[ $(uname -m) = arm64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-darwin-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/
```

For other platforms, download the appropriate binary from the [KinD releases page](https://github.com/kubernetes-sigs/kind/releases) and move it to a directory in your PATH. Verify the installation with:

```sh
kind version
```

## Step 2: Create a KinD Cluster

Create a multi-node Kubernetes cluster using a configuration file to define the cluster structure. Below is an example configuration for a cluster named `vinhphamtechlab` with one control plane and two worker nodes. Example manifest:

```yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
  - role: control-plane
  - role: worker
  - role: worker
networking:
  apiServerAddress: "0.0.0.0"
  apiServerPort: 6443
  podSubnet: "192.168.0.0/16"
  serviceSubnet: "10.96.0.0/12"
  disableDefaultCNI: true
```

Create the cluster with:

```sh
kind create cluster --config config.yaml --name vinhphamtechlab
```

Verify the cluster is running:

```sh
kubectl cluster-info --context kind-vinhphamtechlab
kubectl get nodes
```

## Step 3: Set Up a Custom Network with Calico

Calico provides advanced networking and network policy capabilities for Kubernetes. To set up Calico, install the Tigera operator and Calico system components using kustomize with Helm support.

### Install Tigera Operator

The Tigera operator manages Calico's lifecycle. Apply the manifests:

```sh
kustomize build --enable-helm network/tigera-operator | kubectl apply --server-side --force-conflicts -f-
```

This command creates the `tigera-operator` namespace and deploys custom resource definitions (CRDs), service accounts, roles, and the operator deployment.

### Install Calico System

Next, deploy the Calico system components to enable Calico as the cluster's Container Network Interface (CNI):

```sh
kustomize build --enable-helm network/calico-system | kubectl apply --server-side --force-conflicts -f-
```

This sets up the `calico-system` namespace and applies resources like the `Installation` and `APIServer` CRDs, enabling Calico's networking features. Verify the installation:

```sh
kubectl get pods -n calico-system
```

Ensure all pods are in the `Running` state.

## Step 4: Set Up Storage with OpenEBS

OpenEBS provides dynamic storage provisioning for Kubernetes workloads. Install the OpenEBS operator to enable local storage management.

Apply the OpenEBS manifests:

```sh
kustomize build --enable-helm storage/openebs-operator | kubectl apply --server-side --force-conflicts -f-
```

Verify the OpenEBS installation:

```sh
kubectl get pods -n openebs
kubectl get storageclass
```

Ensure all pods are running and the `openebs-hostpath` storage class is available for persistent volume claims (PVCs).

## Step 5: Testing the Setup

To confirm the cluster is functional:

1. **Check Nodes**: Run `kubectl get nodes` to verify the control plane and worker nodes.
2. **Test Networking**: Deploy a sample application (e.g., an NGINX pod) and create a Calico network policy to control traffic.
3. **Test Storage**: Create a PVC using the `openebs-hostpath` storage class and verify it binds to a persistent volume.

Example PVC configuration:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: sample-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: openebs-hostpath
  resources:
    requests:
      storage: 1Gi
```

Apply it with:

```sh
kubectl apply -f sample-pvc.yaml
```

Check the PVC status:

```sh
kubectl get pvc
```

## Conclusion

Using KinD, Calico, and OpenEBS, you can set up a fully functional local Kubernetes cluster with custom networking and storage. KinD's lightweight container-based nodes make it ideal for development, while Calico provides robust network policies and OpenEBS enables dynamic storage provisioning. This setup is perfect for testing Kubernetes features, developing applications, or learning advanced concepts like service meshes or RBAC on a single machine. To tear down the cluster, run:

```sh
kind delete cluster --name vinhphamtechlab
```

This environment empowers developers to experiment with Kubernetes efficiently and cost-effectively.
