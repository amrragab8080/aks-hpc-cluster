# aks-hpc-cluster
In this example we can standup a AKS cluster for running various HPC workloads including deep learning with tensorflow. This will use Azure Kubernetes Service (AKS)
with the kubeflow MPI operator. 

## Create an HPC Cache
Create a HPC Cache in the Azure Portal according to the Azure documentation.

## Create AKS Cluster
Create a AKS cluster in the same subnet as the HPC Cache created earlier. We are using the cluster autoscaler which will start a single node in the default nodepool.
Once the job is submitted the cluster will expand the nodepool to fit the requirements of the job(s).
```
az aks create --resource-group araks --name araksml --node-count 1 --enable-addons monitoring --generate-ssh-keys --node-vm-size Standard_ND24s --vm-set-type VirtualMachineScaleSets --enable-cluster-autoscaler --min-count 1 --max-count 4 --load-balancer-sku standard --kubernetes-version 1.15.10 --vnet-subnet-id <subnet_hpccache>
az aks get-credentials --resource-group araks --name arakstest
```

## Apply NVIDIA PLugin
```
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.12/nvidia-device-plugin.yml
```
## Apply HPC Cache PV/PVC
