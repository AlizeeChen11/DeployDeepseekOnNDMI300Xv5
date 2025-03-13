# DeployDeepseekOnNDMI300Xv5
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
sudo docker pull rocm/sglang-staging:20250303








