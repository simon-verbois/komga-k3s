# Deploying Komga on Kubernetes

This repository contains a set of Kubernetes manifests for deploying [Komga](https://komga.org/), a free and open source comics/mangas server.

The configuration is designed to separate application configuration, metadata, and the actual media files for better data management.

<br>

## Prerequisites

1. A working Kubernetes cluster (e.g., K3s, k0s, RKE2).
2. The `kubectl` command-line tool configured to access your cluster.
3. An **Ingress Controller** installed in the cluster (e.g., Traefik, NGINX Ingress).
4. A **StorageClass** configured to dynamically provision storage volumes. 
5. A directory on your Kubernetes node(s) containing your comics/mangas library that you can point the deployment to.

<br>

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/simon-verbois/komga-k8s
cd komga-k8s
```

### 2. Rename the files

```bash
for file in *.yaml.template; do mv "$file" "${file%.template}"; done
```

### 3. Custom the configuration files

You have to adapt the yaml file, follow the hint for each file

### 4. Deploy Komga

Apply all YAML manifests in a single command from the project root:

```bash
kubectl apply -f .
```

This will create the `komga` namespace, the PersistentVolumeClaims, the ConfigMap, the Deployment, the Service, and the Ingress.

### 5. Access Komga

After a few moments, the container image will be downloaded and the pod will start. You should then be able to access your Komga instance via the URL you configured in the ingress (e.g., `https://komga.your-domain.com`).

The initial setup, including creating an admin user and adding your libraries, will be done through this web interface.

<br>

## Maintenance

### Updating the Komga Image

The deployment uses the `gotson/komga:latest` image. To update to the latest version, you can trigger a rolling update of the deployment, which will force Kubernetes to pull the newest image.

```bash
kubectl rollout restart deployment/komga-deployment -n komga
```

You can monitor the progress of the update with:

```bash
kubectl rollout status deployment/komga-deployment -n komga
```

<br>

## Uninstallation

To remove all the resources created by these manifests, run the following command:

```bash
kubectl delete -f .
```

**Note:** This will also delete the `komga` namespace. The `PersistentVolumeClaim` will be deleted, but the actual data on your storage volume might remain, depending on your `StorageClass` reclaim policy.

<br>

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

