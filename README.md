# Deploy Deepseek On Azure NDMI300Xv5
Deploy Deepseek R1 locally on Azure NDMI300Xv5 

## Deploy an Azure NDMI300Xv5 VM using marketplace ROCM image:
az vm create --resource-group RG --name VMNAME --image microsoft-dsvm:ubuntu-hpc:2204-rocm:22.04.2025030701 --public-ip-sku Standard --admin-username azureuser --admin-password 123ABC --size Standard_ND96isr_MI300X_v5  --location LOCATION --security-type Standard

## Additional steps
Configure NVMe disk as raid disk
```
sudo sh ./scripts/setup-nvme.sh
```

Configure Hugging Face to use the RAID-0
```
mkdir –p /mnt/resource_nvme/hf_cache 
export HF_HOME=/mnt/resource_nvme/hf_cache
```

Configure Docker to use the RAID-0
```
mkdir -p /mnt/resource_nvme/docker 
sudo tee /etc/docker/daemon.json > /dev/null <<EOF 
{ 
    "data-root": "/mnt/resource_nvme/docker" 
}

EOF 
sudo chmod 0644 /etc/docker/daemon.json 
sudo systemctl restart docker 
```
## Running DeepSeek-R1
```
sudo docker pull rocm/sglang-staging:20250303
sudo docker run --device=/dev/kfd --device=/dev/dri --security-opt seccomp=unconfined --cap-add=SYS_PTRACE --group-add video --privileged --shm-size 32g --ipc=host -p 30000:30000 -v /mnt/resource_nvme:/mnt/resource_nvme -e HF_HOME=/mnt/resource_nvme/hf_cache -e HSA_NO_SCRATCH_RECLAIM=1 -e GPU_FORCE_BLIT_COPY_SIZE=64 -e DEBUG_HIP_BLOCK_SYN=1024 rocm/sglang-staging:20250303 python3 -m sglang.launch_server --model deepseek-ai/DeepSeek-R1 --tp 8 --trust-remote-code --host 0.0.0.0
```
Once you see "The server is fired up and ready to roll!", you can begin making queries to the model. 
```
curl http://localhost:30000/get_model_info
curl http://localhost:30000/generate -H "Content-Type: application/json" -d '{ "text": "Once upon a time,", "sampling_params": { "max_new_tokens": 16, "temperature": 0.6 } }'
```
## Run serving bench:
Run container:
```
sudo docker run -d -it --ipc=host --network=host --privileged --device=/dev/kfd --device=/dev/dri --device=/dev/mem --group-add render --security-opt seccomp=unconfined rocm/sglang-staging:20250303
sudo docker ps
sudo docker exec -it <container id> bash
HSA_NO_SCRATCH_RECLAIM=1 python3 -m sglang.launch_server --model deepseek-ai/DeepSeek-R1 --tp 8 --trust-remote-code
```
```
concurrency_values=(128 64 32 16 8 4 2 1)

for concurrency in "${concurrency_values[@]}";do
    python3 -m sglang.launch_server \
    --dataset-name random \
    --random-range-ratio 1 \
    --num-prompt 500 \
    --random-input 3200 \
    --random-output 800 \
    --max-concurrency "${concurrency}"
done
```
## Reference:

https://techcommunity.microsoft.com/blog/AzureHighPerformanceComputingBlog/running-deepseek-r1-on-a-single-ndv5-mi300x-vm/4372726
https://rocm.blogs.amd.com/artificial-intelligence/DeepSeekR1_Perf/README.html














