# MySQL Deployment Using ConfigMap and Secret (Without PV and PVC)

## Project Overview

This project demonstrates how to deploy MySQL using Kubernetes StatefulSet, ConfigMap, Secret, and Headless Service without PersistentVolume (PV) and PersistentVolumeClaim (PVC).

### Components Used

- MySQL 8.0
- ConfigMap – Stores the database name.
- Secret – Stores the MySQL root password.
- StatefulSet – Deploys three MySQL replicas.
- Headless Service – Provides stable network identities to StatefulSet pods.

## Project Structure

```text
.
├── namespace.yml
├── config.yml
├── secret.yml
├── service.yml
├── statefulset.yml
└── README.md
```

## 1. Create Namespace

```bash
kubectl apply -f namespace.yml
```

## 2. Create ConfigMap

The ConfigMap stores the MySQL database name.

```yaml
data:
  MYSQL_DATABASE: devops
```

Apply:

```bash
kubectl apply -f config.yml
```

## 3. Create Secret

The Secret stores the MySQL root password.

```bash
kubectl apply -f secret.yml
```

## 4. Create Headless Service

```bash
kubectl apply -f service.yml
```

## 5. Deploy MySQL StatefulSet

```bash
kubectl apply -f statefulset.yml
```

The StatefulSet creates three MySQL pods.

## 6. Verify Deployment

Check pods:

```bash
kubectl get pods -n mysql
```

Check services:

```bash
kubectl get svc -n mysql
```

Check ConfigMap:

```bash
kubectl get configmap -n mysql
```

Check Secret:

```bash
kubectl get secret -n mysql
```

Check PVCs:

```bash
kubectl get pvc -n mysql
```

No PVCs will be created in this project.

## 7. Connect to MySQL

Connect to the first MySQL pod:

```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- mysql -u root -p
```

Enter the password configured in `secret.yml`.

Show databases:

```sql
SHOW DATABASES;
```

Exit MySQL:

```sql
exit;
```

## 8. Update the Database Name Using ConfigMap

To change the database name, modify `config.yml`.

Initially:

```yaml
data:
  MYSQL_DATABASE: devops
```

Change it to:

```yaml
data:
  MYSQL_DATABASE: hii
```

Apply the updated ConfigMap:

```bash
kubectl apply -f config.yml
```

### Check Environment Variable

Before restarting the pods:

```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- printenv MYSQL_DATABASE
```

The old value will still be displayed because environment variables are not automatically updated in existing containers.

### Restart StatefulSet

```bash
kubectl rollout restart statefulset mysql-statefulset -n mysql
```

Wait for rollout completion:

```bash
kubectl rollout status statefulset mysql-statefulset -n mysql
```

Verify the updated environment variable:

```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- printenv MYSQL_DATABASE
```

Expected output:

```text
hii
```

### Verify Updated Database

Connect again:

```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- mysql -u root -p
```

Execute:

```sql
SHOW DATABASES;
```

The newly initialized MySQL instance should contain the `hii` database.

## 9. Apply All YAML Files Together

To deploy all resources:

```bash
kubectl apply -f .
```

To verify:

```bash
kubectl get all -n mysql
```

## 10. Delete Resources

Delete the StatefulSet:

```bash
kubectl delete -f statefulset.yml
```

Delete the Service:

```bash
kubectl delete -f service.yml
```

Delete ConfigMap and Secret:

```bash
kubectl delete -f config.yml
kubectl delete -f secret.yml
```

Delete Namespace:

```bash
kubectl delete -f namespace.yml
```

## Important Notes

- This project does not use PV or PVC.
- MySQL stores data in the container's writable filesystem.
- Updating ConfigMap does not automatically update environment variables in running pods.
- Restarting the StatefulSet allows new pods to receive updated environment variables.
- Without persistent storage, replacing a MySQL container causes its existing database data to be lost.
- `MYSQL_DATABASE` is used during the initial MySQL database initialization.
- Each StatefulSet replica maintains its own independent database. There is no automatic database replication between the three replicas.

**Learning Outcome:** Understanding ConfigMap, Secret, StatefulSet, Headless Service, environment variable updates, and MySQL initialization in Kubernetes.
