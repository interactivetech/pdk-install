# Step 8: Install Determined, Pipeline Secrets, and KServe Parameters

This step focuses on installing the Determined AI platform, configuring pipeline secrets for integration with Pachyderm and KServe, and setting up KServe permissions.

## Install Determined

1. **Download and Extract Determined**:
   Download the Determined package and extract it:

   ```bash
   wget https://hpe-mlde.determined.ai/latest/_downloads/389266101877e29ab82805a88a6fc4a6/determined-latest.tgz
   tar xvf determined-latest.tgz
   ```
  Configure values.yaml:
  
# Note: 
we are defining `/mnt/efs/shared_fs/` as our shared file space
Edit the values.yaml file within the Determined folder to match the following configuration, adjusting the database and checkpoint storage settings:
```yaml
db:
  hostAddress: "postgres-service.default.svc.cluster.local."
  name: determined
  user: postgres
  password: "demo"
  port: 5432
checkpointStorage:
  saveExperimentBest: 0
  saveTrialBest: 1
  saveTrialLatest: 1
  type: shared_fs
  hostPath: /mnt/efs/shared_fs/determined
```

Install Determined:
Use Helm to install Determined:

```bash
helm install determinedai ./determined
```
Configure Secrets and Permissions
Set Environment Variables:
Set the MLDE and MLDM host IP addresses:
```bash
export MLDE_HOST=10.239.255.103
export MLDM_HOST=10.239.255.101
```

Create Pipeline Secrets:
Create a Kubernetes secret to store pipeline configuration:
```bash
cat <<EOF > "./pipeline-secret.yaml"
apiVersion: v1
kind: Secret
metadata:
  name: pipeline-secret
  namespace: ${MLDM_NAMESPACE}
stringData:
  det_master: "${MLDE_HOST}:8080"
  det_user: "admin"
  det_password: ""
  pac_token: ""
  pachd_lb_service_host: "${MLDM_HOST}"
  pachd_lb_service_port: "9090"
  kserve_namespace: "${KSERVE_MODELS_NAMESPACE}"
EOF

kubectl -n ${MLDM_NAMESPACE} apply -f pipeline-secret.yaml
```

Configure KServe Permissions:
Apply the necessary roles and bindings for KServe integration:
```bash
cat <<EOF > "./mldm-kserve-perms.yaml"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kserve-inf-service-role
  namespace: ${KSERVE_MODELS_NAMESPACE}
  labels:
    app: kserve-inf-app
rules:
- apiGroups: ["serving.kserve.io"]
  resources: ["inferenceservices"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kserve-inf-service-role
subjects:
- kind: ServiceAccount
  name: pachyderm-worker
  namespace: ${MLDM_NAMESPACE}
EOF

kubectl apply -f mldm-kserve-perms.yaml
```

Save Configuration to ConfigMap
Ingress Host and Port:
Retrieve the ingress host and port for KServe:
```bash
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'); echo $INGRESS_HOST
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}'); echo $INGRESS_PORT
```

Create ConfigMap:
Store the PDK and related services configuration in a ConfigMap:
```bash
cat <<EOF > ./pdk-config.yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: pdk-config
  namespace: default
data:
  region: "on-prem"
  mldm_namespace: "${MLDM_NAMESPACE}"
  mldm_bucket_name: "on-prem"
  mldm_host: "${MLDM_HOST}"
  mldm_port: "9090"
  mldm_url: "http://${MLDM_HOST}:9090"
  mldm_pipeline_secret: "pipeline-secret"
  mlde_bucket_name: "on-prem"
  mlde_host: "${MLDE_HOST}"
  mlde_port: "80"
  mlde_url: "http://${MLDE_HOST}:80"
  kserve_model_bucket_name: "on-prem"
  kserve_model_namespace: "${KSERVE_MODELS_NAMESPACE}"
  kserve_ingress_host: "${INGRESS_HOST}"
  kserve_ingress_port: "${INGRESS_PORT}"
  kserve_service_hostname: "${SERVICE_HOSTNAME}"
  db_connection_string: ${DB_CONNECTION_STRING}
  registry_uri: "gcr.io/dai-dev-554"
  pdk_name: "${NAME}"
EOF

kubectl apply -f ./pdk-config.yaml
```

Create Service Account for KServe
Create a service account and secret for KServe to access Pachyderm:
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: pach-kserve-creds
  namespace: ${KSERVE_MODELS_NAMESPACE}
  annotations:
    serving.kserve.io/s3-endpoint: pachd.${MLDM_NAMESPACE}:30600
    serving.kserve.io/s3-usehttps: "0"
type: Opaque
stringData:
  AWS_ACCESS_KEY_ID: "dummycredentials"
  AWS_SECRET_ACCESS_KEY: "dummycredentials"
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pach-deploy
  namespace: ${KSERVE_MODELS_NAMESPACE}
  annotations:
    serving.kserve.io/s3-endpoint: pachd.${MLDM_NAMESPACE}:30600
    serving.kserve.io/s3-usehttps: "0"
secrets:
- name: pach-kserve-creds
EOF
```

Set Up Determined
Log in to Determined and configure templates:
```bash
export DET_MASTER=10.239.255.103:8080
det user login admin
det template set llm-rag-env-mistral llm-rag-env-mistral.yaml
```
This concludes the setup for Determined, including configuring pipeline secrets for integration and setting up permissions for KServe within your Kubernetes environment.