# lsc-lab6
Large Scale Computing LAB6


# Lab6 - Kubernetes NFS-backed Web Server

This project demonstrates how to deploy an Nginx-based HTTP server on AWS EKS using a dynamically provisioned NFS volume for shared storage. It includes:

- An Elastic Kubernetes Service (EKS) cluster definition
- Helm chart installation for an in-cluster NFS provisioner
- Kubernetes manifests for PersistentVolumeClaim, Deployment, Service, and Job

## Prerequisites

- AWS account with permissions to create EKS clusters and Load Balancers
- AWS CLI installed and configured (`aws configure`)
- `kubectl` installed
- `helm` installed
- (Optional) `eksctl` if you prefer CLI cluster creation

## Setup and Deployment

1. **Clone the repository**

   ```bash
   git clone https://github.com/Glonk223/lsc-lab6.git
   cd lsc-lab6
   ```

2. **Create your EKS cluster**

   - Via console or using `eksctl`:
     ```bash
     eksctl create cluster \   
       --name lsc-lab6-cluster \
       --region us-east-1 \
       --nodes 2
     ```

3. **Configure kubectl**

   ```bash
   aws eks --region us-east-1 update-kubeconfig --name lsc-lab6-cluster
   ```

4. **Install the NFS Server Provisioner**
   We recommend using the Bitnami chart:

   ```bash
   helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
   helm repo update
   helm install my-release nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner
   ```

5. **Create the PersistentVolumeClaim**

   ```bash
   kubectl apply -f pvc.yaml
   kubectl get pvc nfs-pvc  # should show STATUS=Bound
   ```

6. **Deploy the Nginx web server**

   ```bash
   kubectl apply -f deployment.yaml
   kubectl get pods -l app=web-server  # should show a Running pod
   ```

7. **Expose the Deployment via Load Balancer**

   ```bash
   kubectl apply -f service.yaml
   kubectl get svc web-server-service  # note the EXTERNAL-IP
   ```

8. **Populate sample content**

   ```bash
   kubectl apply -f job.yaml
   kubectl get jobs copy-content-job  # ensure it completes
   ```

9. **Test the server**

   ```bash
   # Replace XXX.XXX.XXX.XXX with the Load Balancer's EXTERNAL-IP
   curl http://XXX.XXX.XXX.XXX
   # or open http://XXX.XXX.XXX.XXX in your browser
   ```

## Cleanup

To tear down and clean up resources:

```bash
kubectl delete -f job.yaml deployment.yaml service.yaml pvc.yaml
helm uninstall nfs-server
eksctl delete cluster --name lsc-lab6-cluster  # if you used eksctl
```
