# Kubernetes Cluster Analysis - 192.168.1.242

## Cluster Overview
**Control Plane**: https://127.0.0.1:6443
**CoreDNS**: Running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
**Metrics Server**: Running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

## Node Information
| NAME | STATUS | ROLES | AGE | VERSION | INTERNAL-IP | OS-IMAGE | KERNEL-VERSION | CONTAINER-RUNTIME |
|------|--------|-------|-----|---------|-------------|----------|-----------------|-------------------|
| kubernetes-1 | Ready | control-plane,master | 27h | v1.33.4+k3s1 | 192.168.1.242 | Debian GNU/Linux 13 (trixie) | 6.12.43+deb13-amd64 | containerd://2.0.5-k3s2 |

## Namespaces
- cloudflare (Active, 166m)
- default (Active, 27h)
- home-assistant (Active, 27h)
- kube-node-lease (Active, 27h)
- kube-public (Active, 27h)
- kube-system (Active, 27h)
- longhorn-system (Active, 25h)

## Deployments

### Home Assistant Deployment
**Namespace**: home-assistant
**Name**: home-assistant
**Replicas**: 1/1/1/1
**Strategy**: RollingUpdate (25% max unavailable, 25% max surge)
**Image**: ghcr.io/home-assistant/home-assistant:stable
**Port**: 8123/TCP
**Environment**: TZ=Europe/Stockholm
**Volume**: config (PVC: home-assistant-pvc)
**Service**: home-assistant-service (NodePort: 30080)

### Core System Deployments (kube-system)
- **coredns**: 1 replica, Image: rancher/mirrored-coredns-coredns:1.12.3
- **local-path-provisioner**: 1 replica, Image: rancher/local-path-provisioner:v0.0.31
- **metrics-server**: 1 replica, Image: rancher/mirrored-metrics-server:v0.8.0

### Longhorn Storage Deployments (longhorn-system)
- **csi-attacher**: 3 replicas
- **csi-provisioner**: 3 replicas
- **csi-resizer**: 3 replicas
- **csi-snapshotter**: 3 replicas
- **longhorn-driver-deployer**: 1 replica
- **longhorn-ui**: 2 replicas (NodePort: 32654)

## Services

### Application Services
- **home-assistant-service** (home-assistant): NodePort 8123:30080/TCP
- **kubernetes** (default): ClusterIP 10.43.0.1:443/TCP
- **kube-dns** (kube-system): ClusterIP 10.43.0.10:53/UDP,TCP + 9153/TCP
- **metrics-server** (kube-system): ClusterIP 10.43.34.66:443/TCP

### Longhorn Services
- **longhorn-admission-webhook**: ClusterIP 10.43.197.85:9502/TCP
- **longhorn-backend**: ClusterIP 10.43.81.3:9500/TCP
- **longhorn-conversion-webhook**: ClusterIP 10.43.8.85:9501/TCP
- **longhorn-frontend**: NodePort 80:32654/TCP
- **longhorn-recovery-backend**: ClusterIP 10.43.3.226:9503/TCP

## Storage Configuration

### Persistent Volumes
- **pvc-ea4adf23-44df-4e43-afa2-a35e9c54c80c**: 10Gi, RWO, Bound to home-assistant/home-assistant-pvc, longhorn storage class

### Persistent Volume Claims
- **home-assistant-pvc** (home-assistant): 10Gi, RWO, longhorn storage class

### Storage Classes
- **local-path** (default): rancher.io/local-path, Delete, WaitForFirstConsumer
- **longhorn**: driver.longhorn.io, Delete, Immediate, AllowVolumeExpansion: true

## Key Pods

### Home Assistant Pod
- **Name**: home-assistant-cb669d76c-lq827
- **Status**: Running
- **IP**: 10.42.0.37
- **Node**: kubernetes-1
- **Image**: ghcr.io/home-assistant/home-assistant:stable@sha256:89ec0583c7f47c8a150204f6b5ed48b5432026012bebe1226cf72775a795a5e1

### CoreDNS Pod
- **Name**: coredns-64fd4b4794-r7grl
- **Status**: Running
- **IP**: 10.42.0.3
- **Priority**: 2000000000 (system-cluster-critical)

### Longhorn Manager Pod
- **Name**: longhorn-manager-wn7fn
- **Status**: Running
- **IP**: 10.42.0.13
- **Priority**: 1000000000 (longhorn-critical)
- **Image**: longhornio/longhorn-manager:v1.6.1

## Architecture Summary

This is a **K3s single-node cluster** running on Debian with the following key characteristics:

1. **Control Plane**: Single master node (kubernetes-1)
2. **Container Runtime**: containerd 2.0.5-k3s2
3. **Networking**: K3s built-in networking with CoreDNS
4. **Storage**: Longhorn distributed block storage system
5. **Ingress**: NodePort services exposed on ports 30080 (Home Assistant) and 32654 (Longhorn UI)
6. **Monitoring**: Metrics Server enabled for resource monitoring

## Key Applications

### Home Assistant
- **Access**: http://192.168.1.242:30080
- **Storage**: 10Gi Longhorn persistent volume
- **Configuration**: Mounted to /config with Europe/Stockholm timezone

### Longhorn Storage UI
- **Access**: http://192.168.1.242:32654
- **Purpose**: Web interface for managing distributed block storage

## Security Notes
- All kube-root-ca.crt configmaps are present across namespaces
- Longhorn webhook TLS certificates configured
- K3s serving certificates in place
- Service account tokens properly configured

## Cluster Health
- All system pods running
- No failed pods or events
- Storage system operational
- DNS resolution functional
- Metrics collection active

This cluster is ready for production use with proper storage, monitoring, and application deployment capabilities.