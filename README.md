# Simple VM Cronjob in Azure

*login to your azure portal subscription from terminal*
```bash
az login --use-device-login
```

*create/ssh test resource group and virtual machine*
```bash
RG_NAME=rg_test && \
VM_NAME=vm_test && \
USERNAME=azureuser && \
PASSWORD=<PASSWORD> && \
LOCATION=eastus && \
IMAGE=$(az vm image list --publisher Canonical --query "[0].urn" --output tsv)
SIZE=Standard_B1ls
az group create \
  --location $LOCATION \
  --name $RG_NAME && \
az vm create \
  --location $LOCATION \
  --resource-group $RG_NAME \
  --name $VM_NAME \
  --image $IMAGE \
  --size $SIZE \
  --admin-username azureuser \
  --admin-password $PASSWORD && \
PUBLIC_IP=$(az vm show -d -g $RG_NAME -n $VM_NAME --query publicIps -o tsv) && \
ssh $USERNAME@$PUBLIC_IP
```

*creates a cronjob that writes the datetime everyminute to a text file*
```bash
echo '* * * * * echo $(date) >> ~/logfile.txt' | crontab -
```
