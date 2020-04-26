# aks-hpc-cluster
In this example we can standup a AKS cluster for running various HPC workloads including deep learning with tensorflow. This will use Azure Kubernetes Service (AKS)
with the kubeflow MPI operator. 

## Create AKS Cluster
```az aks create --resource-group araks --name araksml --node-count 1 --enable-addons monitoring --generate-ssh-keys --node-vm-size Standard_ND24s --vm-set-type VirtualMachineScaleSets --enable-cluster-autoscaler --min-count 1 --max-count 4 --load-balancer-sku standard --kubernetes-version 1.15.10
az aks get-credentials --resource-group araks --name arakstest```

## Apply NVIDIA PLugin
```kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.12/nvidia-device-plugin.yml```
