# cronjob_simple
simple tutorial for cronjob in azure

```bash
az login --use-device-login
```

```bash
RG_NAME=rg_test && \
VM_NAME=vm_test && \
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
echo $PUBLIC_IP
```
