# Jenkins Master Deployment on Kubernetes Cluster

This guide will walk you through deploying a Jenkins master on a Kubernetes cluster, specifically on the "ET Cluster". The deployment involves setting up a Jenkins master with customized configurations to meet specific requirements.

## Prerequisites

1. **Node Selector**: Ensure that the node selector name is `et-worker-node-13`.
2. **Namespace**: Create a namespace called `ema-dev` in your Kubernetes cluster.
3. **Docker Image Secret**: A Docker image secret must be created for both Jenkins master and slave.
4. **Persistent Volume Claim**: Create an 8GB PVC for backup purposes.

## Configuration Changes

### Modifications in `values.yaml`

1. **Node Selector**: Update the node selector in two places:
   ```yaml
   nodeSelector:
     kubernetes.io/hostname: et-worker-node-13
   ```

2. **Image Pull Secret**: Configure the image pull secret for Jenkins master and slave:
   ```yaml
   imagePullSecretName: emadevops-acr-secret
   jnlpregistry:
     image:
       repository: "emadevops.azurecr.io/jenkins-slave"
       tag: "1.0"
     workingDir: "/home/jenkins/agent"
     nodeUsageMode: "NORMAL"
     customJenkinsLabels: []
     imagePullSecretName: emadevops-acr-secret
   ```

3. **Jenkins Master Docker Image**:
   ```yaml
   controller:
     componentName: "jenkins-controller"
     image:
       registry: "docker.io"
       repository: "jenkins/jenkins"
       tagLabel: jdk17
       pullPolicy: "Always"
     imagePullSecretName: emadevops-acr-secret
   ```

4. **Service Type**: Change the service type to `NodePort`.

## Deployment Steps

1. **Create Docker Image Secret**:
   ```bash
   kubectl create secret docker-registry emadevops-acr-secret \
     --docker-server=emadevops.azurecr.io \
     --docker-username=emadevops \
     --docker-password=YOUR_DOCKER_PASSWORD \
     --docker-email=sravan.kumar.pulle@pwc.com \
     --namespace=ema-dev
   ```
   ```bash
   kubectl create secret docker-registry emadevops-acr-secret   --docker-server=emadevops.azurecr.io   --docker-username=emadevops   --docker-password=--docker-email=sravan.kumar.pulle@pwc.com   --namespace=ema-dev
   ```

2. **Add Jenkins Helm Repository**:
   ```bash
   helm repo add jenkins https://charts.jenkins.io
   ```

3. **Download and Untar Jenkins Chart**:
   ```bash
   helm pull jenkins/jenkins --untar
   cd jenkins
   ```

4. **Customize `values.yaml`**: Edit the `values.yaml` file based on the configuration changes mentioned above.

5. **Install Jenkins Master**:
   ```bash
   helm install jenkins-master-ema-devops ./jenkins/ -n ema-dev
   ```

6. **Verify Installation**:
   ```bash
   helm list -n ema-dev
   ```

7. **Upgrade Jenkins Master (if needed)**:
   ```bash
   helm upgrade jenkins-master-ema-devops ./jenkins/ -n ema-dev
   ```

## Access Jenkins UI

- Once the Jenkins UI is up, the default username is `admin`.
- To retrieve the Jenkins admin password, execute the following command:
  ```bash
  kubectl exec --namespace ema-dev -it svc/jenkins-master-ema-devops -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo
  ```

Ensure that you replace `YOUR_DOCKER_PASSWORD` with your actual Docker password in the appropriate command. Follow the above steps to successfully deploy and configure Jenkins master on your Kubernetes cluster.
