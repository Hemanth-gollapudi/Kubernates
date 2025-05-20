# EKS Cluster and Postgres Deployment

## Overview

This project demonstrates how to create an Amazon EKS cluster, deploy a Postgres database, and verify data persistence.

## Prerequisites

- AWS CLI installed and configured (`aws configure`)
- `kubectl` installed
- `eksctl` installed

## Step 1: Create an EKS Cluster

### Install eksctl

- **Windows (using Chocolatey):**
  ```powershell
  choco install eksctl
  ```
- **macOS (using Homebrew):**
  ```bash
  brew install eksctl
  ```
- **Linux (using Binary Release):**
  ```bash
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  sudo mv /tmp/eksctl /usr/local/bin
  ```

### Create the EKS Cluster

```bash
eksctl create cluster \
  --name my-eks-cluster \
  --region us-west-2 \
  --nodegroup-name linux-nodes \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

### Update kubeconfig

```bash
aws eks --region us-west-2 update-kubeconfig --name my-eks-cluster
```

## Step 2: Deploy Postgres Database

### Create Kubernetes Secret

Create a file named `postgres-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  POSTGRES_DB: bXlkYg==
  POSTGRES_USER: bXl1c2Vy
  POSTGRES_PASSWORD: bXlwYXNzd29yZA==
```

Apply the secret:

```bash
kubectl apply -f postgres-secret.yaml
```

### Deploy Postgres

Create a file named `postgres-deployment.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
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
          image: postgres:15
          env:
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_DB
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_PASSWORD
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: postgres
```

Apply the deployment:

```bash
kubectl apply -f postgres-deployment.yaml
```

## Step 3: Verify Postgres is Working

### Get the Pod Name

```bash
kubectl get pods -l app=postgres
```

### Connect to the Postgres Pod

```bash
kubectl exec -it <postgres-pod-name> -- psql -U myuser -d mydb
```

### Create a Table and Insert Data

At the `psql` prompt, run:

```sql
CREATE TABLE test (id SERIAL PRIMARY KEY, name TEXT);
INSERT INTO test (name) VALUES ('hello world');
SELECT * FROM test;
```

## Step 4: Test Data Persistence

### Delete the Pod

```bash
kubectl delete pod <postgres-pod-name>
```

### Get the New Pod Name

```bash
kubectl get pods -l app=postgres
```

### Connect Again

```bash
kubectl exec -it <new-postgres-pod-name> -- psql -U myuser -d mydb
```

### Verify the Data

```sql
SELECT * FROM test;
```

## Step 5: Clean Up (Optional)

### Delete Resources

```bash
kubectl delete -f postgres-deployment.yaml
kubectl delete -f postgres-secret.yaml
eksctl delete cluster --name my-eks-cluster --region us-west-2
```

## Summary

1. **Created an EKS cluster and node group.**
2. **Deployed a Postgres database with persistent storage and a Kubernetes secret.**
3. **Verified Postgres is running by connecting to it from within the pod.**
4. **Created a table, inserted a row, and verified the data.**
5. **Deleted the pod to simulate a restart, then verified the data persisted.**
