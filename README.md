# Deploying Komga on K3s/Kubernetes

This repository contains a set of Kubernetes manifests for deploying [Komga](https://komga.org/), a free and open source comics/mangas server, on a K3s cluster (tested on a single-node K3s) or any other Kubernetes cluster.

The configuration is designed to be simple, clean, and easily maintainable.

## Features

- Simple deployment via `kubectl apply`.
- Data persistence for configuration and database managed by `PersistentVolumeClaim`s.
- Direct access to your media library via a `hostPath` volume mount.
- Centralized configuration in a `ConfigMap`.
- Secure exposure via an `Ingress` (tested with Traefik).

## Prerequisites

1. A working Kubernetes cluster (e.g., K3s, k0s, RKE2).
2. The `kubectl` command-line tool configured to access your cluster.
3. An **Ingress Controller** installed in the cluster (e.g., Traefik, NGINX Ingress).
4. A **StorageClass** configured to dynamically provision storage volumes. K3s includes `local-path-provisioner`, which works for this configuration.
5. A directory on your Kubernetes node(s) containing your comics/mangas library that you can point the deployment to.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/simon-verbois/komga-k3s
cd komga-k3s
```

### 2. Copy the files

It's a good practice to use the provided files as templates. Make your own copy to edit with this command.

```bash
for file in *.yaml.template; do mv "$file" "${file%.template}"; done
```

### 3. Customize the configuration

You **must** adapt some files to your own environment before applying them.

- **`02-configmap.yaml`**:
    - Change the time zone `TZ` if needed (e.g., `"America/New_York"`).
    - Adjust `JAVA_TOOL_OPTIONS` if you want to allocate more or less memory to the Komga server.

    ```yaml
    data:
      TZ: "Europe/Paris" # <-- EDIT THIS
      JAVA_TOOL_OPTIONS: "-Xmx4g" # <-- EDIT THIS (e.g., -Xmx2g for 2GB)
    ```

<br>

- **`03-deployment.yaml`**:
    - **User/Group Permissions**: Adjust `runAsUser`, `runAsGroup`, and `fsGroup` in `securityContext` to match the user/group ID that owns your media files on the host node. This is critical to avoid permission issues.
    - **Media Library Path**: Change the `hostPath.path` for the `mangas` volume to the absolute path of your comics/mangas library on your Kubernetes node.

    ```yaml
    # ...
    spec:
      # ...
      template:
        # ...
        spec:
          securityContext:
            runAsUser: 1003   # <-- EDIT THIS to your user's UID
            runAsGroup: 1003  # <-- EDIT THIS to your user's GID
            fsGroup: 1003     # <-- EDIT THIS to your user's GID
          # ...
          volumes:
          - name: mangas
            hostPath:
              path: /data/md0/media/mangas # <-- EDIT THIS to your media library path
              type: Directory
          # ...
    ```

<br>

- **`05-ingress.yaml`**:
    - Modify the `host` to use your own domain name.
    - Adapt the ingress `annotations` for your Ingress Controller. The example uses Traefik and assumes your TLS certificate resolver is named `ovhresolver`. Change this to match your setup.

    ```yaml
    metadata:
      # ...
      annotations:
        traefik.ingress.kubernetes.io/router.tls.certresolver: your-certresolver-name # <-- EDIT THIS
    spec:
      rules:
      - host: "komga.your-domain.com" # <-- EDIT THIS
        # ...
      tls:
      - hosts:
        - "komga.your-domain.com" # <-- EDIT THIS
    ```

### 4. Deploy Komga

Apply all YAML manifests in a single command from the project root:

```bash
kubectl apply -f .
```

This will create the `komga` namespace, the PersistentVolumeClaims, the ConfigMap, the Deployment, the Service, and the Ingress.

### 5. Access Komga

After a few moments, the container image will be downloaded and the pod will start. You should then be able to access your Komga instance via the URL you configured in the ingress (e.g., `https://komga.your-domain.com`).

The initial setup, including creating an admin user and adding your libraries, will be done through this web interface.

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

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

