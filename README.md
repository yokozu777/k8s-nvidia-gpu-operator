# NVIDIA GPU Operator (Yandex Cloud)

Helm chart for NVIDIA GPU Operator on **Ubuntu 20.04** nodes in Yandex Cloud.

NVIDIA driver tags: https://catalog.ngc.nvidia.com/orgs/nvidia/-/containers/driver/-/tags?pageSize=100&page=3

## Versions

| Component | Version |
|-----------|---------|
| Helm chart / appVersion | **v24.9.2** |
| node-feature-discovery (dependency) | **v0.16.6** |

### Images (`values.yaml`, registry `docker.io/yokozu`)

| Image | Tag |
|-------|-----|
| gpu-operator | v24.9.2 |
| gpu-operator-validator | v24.9.2 |
| driver | 550.144.03 |
| container-toolkit | v1.17.4-ubuntu20.04 |
| k8s-device-plugin | v0.17.0 |
| k8s-mig-manager | v0.10.0-ubuntu20.04 |
| k8s-driver-manager | v0.7.0 |
| dcgm-exporter | 3.3.9-3.6.1-ubuntu22.04 |
| cuda (init) | 12.6.3-base-ubi9 |
| vgpu-device-manager | v0.2.8 |
| kubevirt-gpu-device-plugin | v1.2.10 |

| Image (`nvcr.io/nvidia/cloud-native`) | Tag |
|----------------------------------------|-----|
| dcgm | 3.3.9-1-ubuntu22.04 |
| nvidia-fs | 2.20.5 |
| gdrdrv | v2.4.1-2 |
| k8s-kata-manager | v0.2.2 |
| k8s-cc-manager | v0.1.1 |

## Prepare the GPU node

Replace `GPU-WORKER` with your node name:

```bash
kubectl taint node GPU-WORKER nvidia.com/gpu=true:NoSchedule
kubectl label node GPU-WORKER nvidia.com/gpu=true --overwrite
```

## Install

```bash
helm upgrade --install gpu-operator . \
  --namespace gpu-operator \
  --create-namespace \
  -f values.yaml
```

## Deployment example

Schedule pods only on GPU nodes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-workload
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gpu-workload
  template:
    metadata:
      labels:
        app: gpu-workload
    spec:
      nodeSelector:
        nvidia.com/gpu: 'true'
      tolerations:
        - key: nvidia.com/gpu
          operator: Equal
          value: 'true'
          effect: NoSchedule
      containers:
        - name: app
          image: nvidia/cuda:12.0.0-base-ubuntu20.04
          resources:
            limits:
              nvidia.com/gpu: 1
```
