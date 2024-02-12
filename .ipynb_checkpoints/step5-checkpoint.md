# Step 5: Environment Configuration and Kubernetes Deployment for PostgreSQL

This step involves setting up environment variables, creating namespaces, and deploying PostgreSQL on Kubernetes with persistent storage and proper configuration. Follow the steps below to set up your environment and deploy PostgreSQL.

## Set Environment Variables

Start by setting up the environment variables needed for the deployment:
These are example values to add, this can be changed to your preferences.
```bash
# can change
`export NAME="andrew-pdk"`
# should not change
`export DB_CONNECTION_STRING="postgres-service.default.svc.cluster.local."` 
# can change
export DB_ADMIN_PASSWORD="demo"
# should not change
export MLDM_NAMESPACE="pachyderm"
# can change, but easier to leave unchanged.
export KSERVE_MODELS_NAMESPACE="models"
```
# Create Namespace for KServe Models
Create a Kubernetes namespace for the KServe models:
```
kubectl create ns $KSERVE_MODELS_NAMESPACE
```
# Prepare Shared folder for MLDM and MLDE
We are making `/mnt/efs/shared_fs/` are shared file space. 

```bash
mkdir -p /mnt/efs/shared_fs
mkdir /mnt/efs/shared_fs/pachyderm
mkdir /mnt/efs/shared_fs/pachyderm/determined
mkdir /mnt/efs/shared_fs/pv1
mkdir /mnt/efs/shared_fs/pv2

## to ensure pods, checkpoints, and pipeline will have access to files
sudo find /mnt/efs/shared_fs \( -type d -exec sudo chmod u+rwx,g+rwx,o+rx {} \; -o -type f -exec sudo chmod u+rw,g+rw,o+r {} \; \)
chmod -R a+rwx /mnt/ -v
```
# Create Storage Class

Create a storage class that will be used for persistent storage:
```bash
kubectl apply -f - <<EOF
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: manual
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: microk8s.io/hostpath
reclaimPolicy: Delete
parameters:
  pvDir: /mnt/efs/shared_fs
volumeBindingMode: WaitForFirstConsumer
EOF
```

Change the default storage class:
```bash
kubectl annotate sc microk8s-hostpath --overwrite=true storageclass.kubernetes.io/is-default-class=false
```

# Persistent Volume and Claim
Define and create a persistent volume and a claim for PostgreSQL:
```bash
# Persistent Volume
kubectl apply -f - <<EOF
kind: PersistentVolume
apiVersion: v1
metadata:
  name: postgres-pv
  labels:
    app: postgres
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/efs/shared_fs/data"
EOF

# Persistent Volume Claim
kubectl apply -f - <<EOF
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF
```

# Config Map for PostgreSQL Configuration
Create a config map to store the database configuration:
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
 name: postgres-configuration
 labels:
   app: postgres
data:
 POSTGRES_DB: postgres
 POSTGRES_USER: postgres
 POSTGRES_PASSWORD: ${DB_ADMIN_PASSWORD}
EOF
```

# Deploy PostgreSQL StatefulSet
Deploy PostgreSQL using a StatefulSet to ensure data persistence:
```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-statefulset
  labels:
    app: postgres
spec:
  serviceName: "postgres"
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        envFrom:
        - configMapRef:
            name: postgres-configuration
        ports:
        - containerPort: 5432
          name: postgresdb
        volumeMounts:
        - name: pv-data
          mountPath: /mnt/efs/shared_fs/postgres/
      volumes:
      - name: pv-data
        persistentVolumeClaim:
          claimName: postgres-pv-claim
EOF
```

# Create PostgreSQL Service
Define and create a service to expose PostgreSQL:
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  labels:
    app: postgres
spec:
  ports:
  - port: 5432
    name: postgres
  type: NodePort
  selector:
    app: postgres
EOF
```

This completes the setup and deployment of PostgreSQL in your Kubernetes environment. Ensure that you have the necessary persistent volumes and configurations in place for a successful deployment.

# Step 6: Configure PostgreSQL with Databases

After deploying PostgreSQL, the next step is to configure it by creating the required databases. This involves connecting to the PostgreSQL service, creating databases, and granting privileges. Follow the steps below to configure your PostgreSQL instance.

## Get the PostgreSQL Service IP

First, you need to find the IP address of the PostgreSQL service to connect to the database. Run the following command to list all services and find the `Cluster-IP` of the `postgres-service`:

```bash
kubectl get svc
```

Look for the postgres-service entry and note the CLUSTER-IP value. This IP address will be used to connect to your PostgreSQL database.

# Connect to PostgreSQL
Use the kubectl run command to start a temporary Pod in the cluster that includes the psql client for connecting to your PostgreSQL service. Replace 10.152.183.50 with the CLUSTER-IP of your postgres-service:
```bash
kubectl run psql -it --rm=true --image=postgres:13 --command -- psql -h <CLUSTER-IP> -U postgres postgres

```

When prompted, enter the password set for the postgres user (as per the environment variable DB_ADMIN_PASSWORD you configured earlier).

# Create Required Databases
Once connected to the PostgreSQL database, execute the following SQL commands to create the necessary databases:
```bash
CREATE DATABASE pachyderm;
CREATE DATABASE dex;
CREATE DATABASE determined;
GRANT ALL PRIVILEGES ON DATABASE pachyderm TO postgres;
GRANT ALL PRIVILEGES ON DATABASE dex TO postgres;
GRANT ALL PRIVILEGES ON DATABASE determined TO postgres;
```
These commands create three databases: pachyderm, dex, and determined. They also grant all privileges on these databases to the postgres user, ensuring that your applications can access and manage these databases as needed.

# Verify Database Creation
To verify that the databases have been created successfully, run the following command in the psql interface:
```bash
\l
```

This command lists all databases in the PostgreSQL server. You should see pachyderm, dex, and determined in the list.

# Exit psql
When you're done, exit the psql interface by typing:
```bash
\q
```

This will terminate the psql session and remove the temporary Pod used for the database connection.

By completing these steps, you've successfully configured PostgreSQL with the required databases for your environment. This setup is crucial for applications that rely on these databases to store and manage their data.