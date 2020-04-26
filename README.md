# aks-hpc-cluster
In this example we can standup a AKS cluster for running various HPC workloads including deep learning with tensorflow. This will use Azure Kubernetes Service (AKS)
with the kubeflow MPI operator. 

## Create an HPC Cache
Create a HPC Cache in the Azure Portal according to the Azure [documentation](https://docs.microsoft.com/en-us/azure/hpc-cache/hpc-cache-create).

## Create AKS Cluster
Create a AKS cluster in the same subnet as the HPC Cache created earlier. We are using the cluster autoscaler which will start a single node in the default nodepool.
Once the job is submitted the cluster will expand the nodepool to fit the requirements of the job(s).
```
az aks create --resource-group araks --name araksml --node-count 1 --enable-addons monitoring --generate-ssh-keys --node-vm-size Standard_ND24s --vm-set-type VirtualMachineScaleSets --enable-cluster-autoscaler --min-count 1 --max-count 4 --load-balancer-sku standard --kubernetes-version 1.15.10 --vnet-subnet-id <subnet_hpccache>
az aks get-credentials --resource-group araks --name araksml
```

## Apply NVIDIA Plugin
```
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.12/nvidia-device-plugin.yml
```
## Apply HPC Cache PV/PVC
Change the IP of the NFS server in the `pv-hpccache-nfs.yaml` file
```
nfs:
    server: 10.240.0.5 
    path: "/hpccache"
```
Apply the hpccache pv/pvc
```
kubectl apply -f pv-hpccache-nfs.yaml
kubectl apply -f pvc-hpccache-nfs.yaml
```
Deploy the Kubeflow mpi-operator on the cluster
```
kubectl apply -f mpi-operator.yaml/deploy/mpi-operator.yaml
```
## Staging data
Staging the data on the HPCCache filesystem is possible through the `staging-data.yaml` modify the configMap path in shtage-data.sh to the container/blob where the data is stored in Azure Blob Storage.
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: stage-data
data:
  stage-data.sh: |
    azcopy cp --recursive "<Blob_URI>" /hpccache
---
```
Start downloading the data from blob onto HPC Cache
```
kubectl create -f stage-data.yaml
kubectl logs -f pod/attach-pvc
```
Follow the logs until download is complete. Once complete we can start the main workflow
```
kubectl create -f tensorflow/tensorflow-example.yaml
```


