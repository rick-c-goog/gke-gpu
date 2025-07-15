# gke-gpu

## 1. Create GKE cluster with GPU nodepool

```
gcloud container clusters create pytorch-training-cluster  \ --num-nodes=2     --zone=us-west1-b  \
  --accelerator="type=nvidia-tesla-t4,count=2,gpu-driver-version=default" \ --machine-type="n1-highmem-2"     --scopes="gke-default,storage-rw"

```

optionally, the GPU drivers can installed manually after GKE cluster or nodepool provisioned:
```
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/master/nvidia-driver-installer/cos/daemonset-preloaded.yaml
```

## 2. Check the NVIDIA device driver and CUDA version installed
```
 cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-version-check
spec:
  restartPolicy: OnFailure
  containers:
  - name: nvidia-version-check
    image: "nvidia/cuda:12.4.0-base-ubuntu22.04"
    command: ["nvidia-smi"]
    resources:
      limits:
         nvidia.com/gpu: "1"
EOF
```
Make sure the device driver and CUDA version, GPU type shows up properly.

```
 kubectl logs nvidia-version-check
```

## 3. Run a testing vector-add GPU workload:
Test run for a simple verctor add:

```
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      image: gcr.io/spectro-images-public/gpu/nvidia/ubuntu-nvidia-add:ubuntu
      # specify gpu request 
      resources:
        limits:
          nvidia.com/gpu: 1
EOF
```

## 4. If the Pod runtime can not load the CUDA libraries, CUDA applications running in Pods consuming NVIDIA GPUs need to dynamically discover CUDA libraries. This requires including /usr/local/nvidia/lib64 in the LD_LIBRARY_PATH environment variable. 
e.g.:
```
containers:
  - name: cuda-vector-add
    image: gcr.io/spectro-images-public/gpu/nvidia/ubuntu-nvidia-add:ubuntu
    env:
    - name: LD_LIBRARY_PATH
      value: "/usr/local/nvidia/lib64"
```
