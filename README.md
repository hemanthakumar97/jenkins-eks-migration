# Jenkins Migration and Configuration Guide

This guide provides step-by-step instructions to set up Jenkins on Kubernetes, configure it, install required plugins, manage backups, and set up authentication.

---

## Prerequisites

- Kubernetes cluster set up and configured
- `kubectl` installed and configured
- Backup of existing Jenkins data

## Step 1: Deploy Jenkins on Kubernetes

### Create a Namespace

```
kubectl create namespace jenkins
```

### Apply the Jenkins Deployment Configuration

Apply the Kubernetes YAML files to deploy Jenkins in the `jenkins` namespace:

```
kubectl apply -f . -n jenkins
```

This will create the necessary resources such as Deployment, PVC, Service Account, Service, and Ingress.

---

## Step 2: Verify Jenkins Pod Status

Retrieve the list of running pods to ensure Jenkins is deployed successfully:

```
kubectl get pods -n jenkins
```

Ensure that the Jenkins pod is listed and running without errors.

---

## Step 3: Retrieve the Initial Admin Password

Jenkins requires an initial admin password for the first login. To retrieve it:

### Access the Jenkins Pod

```
kubectl exec -it <pod_name> -n jenkins -- /bin/bash
```

### Fetch the Admin Password

```
cat /var/jenkins_home/secrets/initialAdminPassword
```

Use this password to log in to the Jenkins web UI.

---

## Step 4: Jenkins UI Setup

1. Open the Jenkins UI in a web browser.
2. Enter the retrieved admin password to unlock Jenkins.
3. Install suggested plugins.
4. Create an admin user.

---

## Step 5: Backup and Restore Jenkins Data

To restore Jenkins from a backup, copy the backup data into the pod while excluding specific configurations.

### Exclude `config.xml`, `users`, and `plugins` Folders

When backing up Jenkins data, it is important to exclude `config.xml`, `users`, and `plugins` folders:

- **config.xml**: Contains Jenkins global configuration, which might not be compatible with a new setup.
- **users folder**: Contains user authentication data, which can cause conflicts when restoring if you are using OAuth for user authentication.
- **plugins folder**: Automatically installed by Jenkins based on system configuration, avoiding compatibility issues.

### Copy Backup Data to Jenkins Pod

```
kubectl cp /home/ec2-user/jenkins-bkp/. <pod_name>:/var/jenkins_home/ --exclude=config.xml --exclude=users --exclude=plugins -n jenkins
```

### Restart Jenkins Pod

```
kubectl delete pod <pod_name> -n jenkins
```

Kubernetes will automatically recreate the pod with the restored data.

---

## Step 6: Configure Jenkins URL

1. Navigate to **Manage Jenkins > System**.
2. Update the **Jenkins URL** field with the correct address.

---

## Step 7 (Optional): Apply Custom Configuration

### If certain configurations are missing after restoring the backup, you can manually copy the `config.xml` file:

```
kubectl cp /<config_path>/config.xml <pod_name>:/var/jenkins_home/config.xml -n jenkins
```

### Restart Jenkins Pod to Apply Configuration

```
kubectl delete pod <pod_name> -n jenkins
```

---

## Step 8: Install Required Jenkins Plugins

1. Navigate to **Manage Jenkins > Plugins**.
2. Install required plugins based on your pipeline and operational needs.
3. Restart Jenkins to ensure the new plugins are applied:

```
kubectl delete pod <pod_name> -n jenkins
```

---

## Step 9: Install Supported Java Version on Slave Nodes

Ensure slave nodes have a compatible Java version installed for Jenkins jobs.

### Install OpenJDK 17 on Slave Nodes

```
sudo yum install java-17-openjdk -y
```

---

## Conclusion

Following these steps ensures a smooth Jenkins migration and configuration on Kubernetes. Regularly monitor your Jenkins setup, keep backups, and update plugins to maintain a stable CI/CD pipeline.