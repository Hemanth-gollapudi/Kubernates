# Common Errors and Troubleshooting

## EKS Cluster Creation

### Error: eksctl not found

- **Cause:** eksctl is not installed or not in your PATH.
- **Solution:** Install eksctl using the instructions in the README.md file.

### Error: Insufficient permissions

- **Cause:** AWS credentials do not have the necessary permissions to create an EKS cluster.
- **Solution:** Ensure your AWS user has the required IAM permissions for EKS.

## Postgres Deployment

### Error: Pod stuck in Pending state

- **Cause:** PersistentVolumeClaim (PVC) is not bound to a PersistentVolume (PV).
- **Solution:** Create a PersistentVolume and ensure it matches the PVC specifications.

#### Commands to Create and Apply postgres-pv.yaml

1. **Create a file named `postgres-pv.yaml` with the following content:**

   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: postgres-pv
   spec:
     capacity:
       storage: 5Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: /mnt/data
   ```

2. **Apply the PersistentVolume:**

   ```bash
   kubectl apply -f postgres-pv.yaml
   ```

3. **Check the PVC status:**
   ```bash
   kubectl get pvc
   ```
   The status should change to `Bound` if the PV is created successfully.

### Error: relation "test" does not exist

- **Cause:** The table was not created in the new pod after a restart.
- **Solution:** Recreate the table manually or use a database migration tool.

### Error: Failed to connect to Postgres

- **Cause:** Postgres pod is not running or the service is not accessible.
- **Solution:** Check the pod status using `kubectl get pods` and ensure the service is correctly configured.

## General Kubernetes Errors

### Error: kubectl command not found

- **Cause:** kubectl is not installed or not in your PATH.
- **Solution:** Install kubectl using the instructions in the README.md file.

### Error: Unable to connect to the server

- **Cause:** kubeconfig is not correctly configured or the cluster is not accessible.
- **Solution:** Update your kubeconfig using `aws eks update-kubeconfig` and ensure your AWS credentials are valid.
