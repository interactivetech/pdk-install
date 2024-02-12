# Step 7: Installing Pachyderm (MLDM)

Pachyderm is a data versioning, data lineage, and data pipeline tool that provides a platform for data science workflows. This step covers installing Pachyderm in your Kubernetes cluster.

## Create Persistent Volumes

Pachyderm requires persistent storage to manage datasets and pipeline states. Start by creating two persistent volumes (PVs):

```bash
# Persistent Volume 1
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/efs/shared_fs/pv/pv1
EOF

# Persistent Volume 2
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/efs/shared_fs/pv/pv2
EOF
```
Configure MLDM
Create a configuration file for MLDM with the necessary settings, including storage backends and security contexts:
```bash
cat <<EOF > mldm_values.yaml
deployTarget: "CUSTOM"
pachd:
  enabled: true
  externalService:
    enabled: true
  storage:
    backend: LOCAL
    local:
      hostPath: /mnt/efs/shared_fs/pachyderm
      requireRoot: true
etcd:
  storageClass: manual
  storageClassName: manual
  size: 10Gi
loki-stack:
  loki:
    securityContext:
      fsGroup: 0
      runAsGroup: 0
      runAsNonRoot: false
      runAsUser: 0
    persistence:
      storageClassName: manual
      size: 10Gi
postgresql:
  enabled: false
global:
  postgresql:
    postgresqlHost: "${DB_CONNECTION_STRING}"
    postgresqlPort: "5432"
    postgresqlSSL: "disable"
    postgresqlUsername: "postgres"
    postgresqlPassword: "${DB_ADMIN_PASSWORD}"
proxy:
  enabled: true
  service:
    type: LoadBalancer
    httpPort: 9090
  tls:
    enabled: false
EOF
```

Install Pachyderm
With the configuration file ready, proceed to install Pachyderm:
```bash
kubectl create ns ${MLDM_NAMESPACE}

helm repo add pachyderm https://helm.pachyderm.com
helm repo update
helm install pachyderm -f ./mldm_values.yaml pachyderm/pachyderm --namespace ${MLDM_NAMESPACE}
```

Verify Installation
Check if the Pachyderm services are running correctly:
```bash
kubectl get svc -A
```

Look for the pachyderm-proxy service to confirm the installation.

# Uninstalling Pachyderm
If you need to uninstall Pachyderm for any reason, follow these commands:
```bash
# Uninstall Pachyderm
sudo microk8s helm3 uninstall pachyderm -n pachyderm
kubectl delete ns pachyderm
kubectl create ns pachyderm
kubectl patch pv pv1 -p '{"spec":{"claimRef": null}}'
kubectl patch pv pv2 -p '{"spec":{"claimRef": null}}'
```
This step completes the installation of Pachyderm, providing a solid foundation for managing your data science workflows and datasets.
