# Azure-AKS-Hello-World-Application


1. Login to Azure Portal

2. Open AZ Cloud shell (Bash) and Create Resource group “myResourceGroup” & AKS Cluster “myAKSCluster” with one node using below commands

    $ az group create --name myResourceGroup --location centralus

    $ az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --node-vm-size Standard_B2s --enable-addons monitoring --generate-ssh-keys

3. (optional) Create new user “aksuser01” in Azure Active Directory (goto Azure Active Directory and select new user)

4. (optional) Assign “Contributor” role to new user on AKS Cluster “myAKSCluster” and required resource groups (goto respective resource group and select IAM)

5. (optional) Login to Azure Portal with new user “aksuser01” and opene cloud shell

6. Connect to the AKS cluster “myAKSCluster“ using commands from the Azure Portal (AKS > myAKSCluster > Connect) and create Pods with “hello-world“ Container conataining “hello-world” image & Load Balance Service “hello-world-load-balancer“ using attached manifest files (download attached files "ajit-hello_world-depl_v2.yml" & "ajit-hello_world-svc_v2.yml" and upload to AZ cloud shell, use below commands)

    $ az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

    $ kubectl get nodes

    $ kubectl get pods

    $ kubectl apply -f ajit-hello_world-depl_v2.yml

    $ kubectl apply -f ajit-hello_world-svc_v2.yml

    $ kubectl get nodes

    $ kubectl get pods

7. Extract external IP from the service “hello-world-load-balancer” and browse the “hello-world” app in new tab svc-ext-ip:8080

    $ kubectl get service hello-world-load-balancer –watch

8. Scaling Pods to 3 and browse the “hello-world” app again in new tab

    $ kubectl scale --replicas=3 deployment/hello-world

9. Scale down Pods to 1

    $ kubectl scale --replicas=1 deployment/hello-world

10. Connect to the pod using SSH

    $ kubectl get pods -o wide

    $ kubectl exec --stdin --tty hello-world-6b58f8d7f-v96ln -- sh

Note: Delete all resources created on Azure as soon as test complete and result shown to avoid utilization charges, new lab can be setup in minutes using above caommads

##    Some more practice for Kubernates resources: 

1. Basic commands for logging to Azure cloud AKS
```  
az login
az account set --subscription xxxxxx
az aks get-credentials --resource-group ajit-rg01 --name ajit-aks-01 --overwrite-existing
```  
2. Test connection to aks
```  
kubectl get nodes
```  
3. Create multi container pod
kubectl create -f ajit-pod.yaml
```  
ajit-pod.yaml
---------------  
apiVersion: v1
kind: Pod
metadata:
  name: ajit-multi-container-pod
  namespace: default
spec: 
  containers:
  - image: nginx
    name: nginx-container
    ports:
    - containerPort: 80
  - image: linuxserver/firefox
    name: firefox-container
    ports:
    - containerPort: 3000
```  

4. Port forwarding
```  
kubectl port-forward pramod-multi-container 3000:3000
```

5. Testing PODS

Open any Browser on your laptop & type http://localhost:3000, then while inside Firefox Browser type below http://localhost:80

You should be able to see Webpage of Firefox Browser and NGINX Browser. This conclude that you are able to communicate between two containers (which are inside a pod) using localhost


6. yaml file for creating two containers in a single POD

```
apiversion: v1
kind: pod
metadata:
    name:myfirstmulticontainerpod
    #namespace:testing
    labels:
      app: webserver
spec:
  containers:
    -image: nginx:latest
     name: nginxcontainer01
     ports:
       - containerPort: 80
    -image: redis:latest
     name: redisconntainer01
     ports:
       - containerPort: 8080
```

7. Terraform code to create a simple  Kubernetes Cluster 

```
data "azurerm_resource_group" "rg" {
  name = "myresourcegroup-rg01"
}

resource "azurerm_kubernetes_cluster" "myakscluster" {
  name                = "myakscluster-01"
  location            = data.azurerm_resource_group.rg.location
  resource_group_name = data.azurerm_resource_group.rg.name
  dns_prefix          = "myakscluster01"

  default_node_pool {
    name              = "k8snodespool"
    min_count         = 1
    max_count         = 3
    vm_size           = "Standard_D2_v2"
    auto_scaling_enabled = "true"
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    network_policy    = "calico"
    load_balancer_sku = "standard"
    
  }

  tags = {
    Environment = "Dev"
  }
}
```
8. yaml file for creating container with volme: 
   **I) emptyDir**
   An emptyDir volume is first created when a Pod is assigned to a Node and initially its empty. All containers in a Pod share use of the emptyDir volume scratch space, such as for a disk-based merge sort. If a container in a Pod crashes the emptyDir content is unaffected. When a Pod is removed from a node for any reason, the data in the emptyDir is deleted forever along with the container.
   *Some uses for an emptyDir are:*
    (1) scratch space, such as for a disk-based merge sort
    (2)checkpointing a long computation for recovery from crashes
    (3) holding files that a content-manager container fetches while a webserver container serves the data
   ```
apiversion: v1
kind: pod
metadata:
    name:myfirstpodwithvol
    #namespace:testing
    labels:
      app: webserver
spec:
  volumes:
  - name: dhondu-vol
    emptyDir: {}
      sizeLimit: 1G
  containers:
    -image: nginx:latest
     name: nginxcontainer01
     ports:
       - containerPort: 80
     volumeMounts:
       - name: dhondu-vol 
         mountPath: /usr/share/ngnix/html      
   ```
      **II) Persistant Volume**
