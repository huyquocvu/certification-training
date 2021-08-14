# Create and Configure Containers

- [Create and Configure Azure Containers](#1)
    - Configure sizing and scaling for Azure Container Instances
    - Configure container groups for Azure Container Instances
- [Create and Configure Azure Kubernetes Service](#2)
    - Configure storage for AKS
    - Configure scaling for AKS
    - Configure network connections for AKS
    - Update an AKS cluster

## Create and Configure Azure Containers {#1}
### What is a container?

| VM | Container |
| - | - |
| ![VM](https://azurecomcdn.azureedge.net/cvt-665f0680393306b6d0a912671748e5cca6b061d55ecee48d4f985eaa8e15e6bf/images/page/overview/what-is-a-container/valprop1.svg) | ![Container](https://azurecomcdn.azureedge.net/cvt-665f0680393306b6d0a912671748e5cca6b061d55ecee48d4f985eaa8e15e6bf/images/page/overview/what-is-a-container/valprop2.svg) |

### Benefits of Container Instances
- Faster startup
- Per second billing
- Security through isolation
- Custom sizes
- Persistent storage
- Linux and Windows

### Core Concepts
- Azure Container Instances allows you to run multiple containers without managing servers
- Image source is the source from which the image is pulled. Can create and upload custom images to your registry
- Registries are the location of images, can be Azure Container Registry, Docker Hub, or other container registry
- Restart policies include **always, on failure**, and **never**.
    - Must be done at the command line with Powershell or Azure CLI

### Creating Azure Container Instances
###### Azure CLI
```Bash
# Create a resource group
az group create --name ps-course-rg --location centralus

# Create and deploy container
az container create --resource-group ps-course-rg --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label az104-demo \
    --ports 80 --restart-policy Always
```
>**NOTE:** always start with creating the resource group

### Azure Container Groups
- Collection of containers on the same host
- Currently supports only Linux container instances
- Deployment options:
    - Resource Manager template
    - YAML file
- Update containers in a group by redeploying the group
- Modified properties that require container deletion:
    - OS type
    - CPU, memory, or GPU
    - Restart policy
    - Network policy

###### Deploying Container Group with ARM template
```Bash
# Create a resource group
az group create --name container-rg --location centralus

# Create and deply Azure Container group from template
az deployment group create --resource-group container-rg --template-file azuredeploy.json
```

###### Second exmaple
```PowerShell
# Create resource group
az group create --name container-rg --location eastus

# Create deployment group based on azuredeploy.json
az deployment group create --resource-group container-rg --template-file azuredeploy.json

# View deployment state
az container show --resource-group container-rg --name demo-container-group --output table

# View container logs
az container logs --resource-group container -rg --name demo-container-group --container-name aci-tutorial-app

# View side-car logs
az container logs --resource-group container-rg -name demo-container-group --container-name aci-tutorial-sidecar
```

---

## Create and Configure Azure Kubernetes Service {#2}
- Configure Storage for AKS
- Configure Scaling for AKS
- Configure network connections for AKS
- Upgrade an AKS cluster

### What is Azure Kubernetes Service (AKS) ?
- Open-source system for automating deployment, scaling, and management of containerized apps
- Container management platform using declarative configuration to orchestrate containers in different compute environments
- Kubernetes deployment is configured as a cluster consisting of at least one master machine and one or more worker machines called nodes or agent nodes
- AKS manages hosted Kubernetes clusters
- AKS cluster master is managed by Azure and is free
    - Only pay for the VMs on which the nodes run

### Create Azure Kubernetes Cluster
- Nodes of the same configuration are grouped into node pools
- When creating a cluster, you create a system node pool
    - This is the initial number of nodes and SKU size defined
- AKS cluster must use virtual machine **scale sets** for the nodes for autoscaling and multiple node pools
- All node pools must reside in the same virtual network
- AKS cluster must use the ***Standard SKU*** load balancer to use multiple node pools

###### Creating an AKS Single Node Cluster with Azure CLI
```PowerShell
# Create a basic single-node AKS cluster
az aks create \
    --resource-group ps-course-rg \
    --name PSAKSCluster \
    --vm-set-type VirtualMachineScaleSets \
    --node-count 2 \
    --generate-ssh-keys \
    --load-balancer-sku standard

# Add an AKS Node Pool
az aks nodepool add \
    --resource-group ps-course-rg \
    --cluster-name PSAKSCluster \
    --name mynodepool \
    --node-count 3
```
*Reference: https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes*

###### Deploying a web app to AKS
```PowerShell
# Need the Kubernetes command-line client. This is already installed in cloud shell
az aks install-cli

# Download credentials and configure kubernetes cli to use them
az ask get-credentials --resource-group ps-course-rg -name PSAKSCluster

# Verify connection
kubectl get notes

# Deploy application using the apply command of YAML manifest
kubectl apply -f azure-vote.yaml

# Test application
kubectl get service azure-vote-front --watch
```
###### azure-vote.yaml
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-back
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-back
        image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
        env:
        - name: ALLOW_EMPTY_PASSWORD
          value: "yes"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  selector:
    matchLabels:
      app: azure-vote-front
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: azure-vote-front
        image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

### Configure Storage for Azure Kubernetes Service
#### Storage Concepts for AKS
| Volumes | Persistant Volumes | Storage Classes | Persistent Volume Claims
| - | - | - | - |
| <ul><li>Represents a way to store, retrieve and persist data across the Pods and through the application lifecycle.</li><li>Can use Azure Disk and Azure Files</li><li>Uses premium or standard storage and are mounted as read-write once and can only be used for a single pod</li><li>Use Azure files if storage needs to be accessed by multiple clusters</li></ul> | <ul><li>Volumes defined and created as part of the Pod lifecycle</li><li>Exists until Pod is deleted</li><li></li></ul> | <ul><li>Used to define tiers of storage</li><li>Can also define the reclaim policy of a Pod</li></ul> | <ul><li>Requests either disk or file storage of a particular storage class </li></ul> |

#### Storage Types and Capabilities
| Use case | Volume plugin | Read/Write once | Read-Only many | Read/Write many | Windows Server container support |
| -------- | ------------- | --------------- | -------------- | --------------- | -------------------------------- |
| **Shared configuration** | Azure Files | Yes | Yes | Yes | Yes |
| **Structured app data** | Azure Disks | Yes | No | Yes | Yes |
| **Unstructured data, File system operations** | BlobFuse | Yes | Yes | Yes | No |
>**Takeaway:** understand how storage needs to be accessed


### Configure Scaling in AKS
#### Scaling Options
- Manually scale pods or nodes
- Horizontal pod auto-scaler (HPA)
    - Kubernetes uses this to monitor demand and automatically scales the number of replicas
    - When configured, guardrails are set by starting the minimum and maximum number of replicas that can run
    - **Cooldown period:** prevents HPA from changing the number of replicas before a previous scale event occurs
- Cluster auto-scaler
    - Auto-scales the **nodes** of the cluster based on compute demand

###### Scaling
```PowerShell
# Manual scale pods
kubectl scale --replicas=5 deployment/azure-vote-front

# Manual scale nodes
az aks scale --resource-group ps-course-rg --name myAKSCluster --node-count 3

# Autoscale
kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
```

### Configure Networking for Azure Kubernetes Service
#### Kubenet and Azure CNI
| Capabilities | Kubenet | Azure CNI |
| ------------ | ------- | --------- |
| Deploy cluster in existing or new vNet | Supported-UDRs manually applied | Supported |
| Pod -> Pod connectivity | Supported | Supported |
| Pod -> VM, VM in same vNet | Works when initiated by pod | Works both ways |
| Pod -> VM, VM in peered vNet | Works when initiated by pod | Works both ways |
| On-prem access using VPN | Works when initiated by pod | Works both ways |
| Access to resources secured by service endpoints | Supported | Supported |
| Expose Kubernetes service using a load balancer, App Gateway, or ingress controller | Supported | Supported |
*Reference: https://bit.ly/2QZvpHz*

#### Kubenet
![Kubenet](https://docs.microsoft.com/en-us/azure/aks/media/use-kubenet/kubenet-overview.png)
- Uses a routing table for inter-pod comms.
- Assigns a CIDR block of unique, resuable IPs
#### Azure CNI
![Azure CNI](https://docs.microsoft.com/en-us/azure/aks/media/concepts-network/advanced-networking-diagram.png)
- Assigns IPs from the subnet to the worker nodes and pods

### Upgrade an AKS Cluster
```PowerShell
# Show current version of AKS
az aks show --resource-group ps-course-rg --name myAKSCluster --output table

# Get available upgrades for the cluster
az aks get-upgrades --resource-group ps-course-rg --name myAKSCluster

# Upgrade cluster
az aks upgrade --resource-group ps-course-rg --name myAKSCluster --kubernetes-version KUBERNETES_VERSION
```
##### Upgrading a cluster
- You can only ugprade one minor version at a time
    - You can upgrade from 1.14.x to 1.15.x
    - You **CANNOT** upgrade from 1.14.x to 1.16.x
>**Remember:** upgrading the AKS cluster is an iterative process
