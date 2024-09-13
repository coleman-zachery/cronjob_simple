# Simple VM Cronjob in Azure

*install aure-cli and login to your azure portal subscription from terminal*
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash && \
az login --use-device-code
```

*create/ssh test resource group and virtual machine*
```bash
RG_NAME=rg_test && \
VM_NAME=vm_test && \
USERNAME=azureuser && \
PASSWORD=<PASSWORD> && \
LOCATION=eastus && \
IMAGE=$(az vm image list --publisher Canonical --query "[0].urn" --output tsv) && \
SIZE=Standard_B1ls && \
az group create \
  --location $LOCATION \
  --name $RG_NAME && \
az vm create \
  --location $LOCATION \
  --resource-group $RG_NAME \
  --name $VM_NAME \
  --image $IMAGE \
  --size $SIZE \
  --admin-username $USERNAME \
  --admin-password $PASSWORD && \
PUBLIC_IP=$(az vm show -d -g $RG_NAME -n $VM_NAME --query publicIps -o tsv) && \
ssh $USERNAME@$PUBLIC_IP
```

*creates a cronjob that writes the datetime every minute to a text file*
```bash
echo '* * * * * echo $(date) >> ~/logfile.txt' | crontab - && \
exit
```

*wait a few minutes before re-logging in to vm*
```bash
ssh $USERNAME@$PUBLIC_IP
```

*display datetime logs*
```bash
cat ~/logfile.txt # should display several datetimes at minute increments && \
exit
```

*stop the vm and clean-up resources*
```bash
az vm stop --resource-group $RG_NAME --name $VM_NAME && \
az vm delete --resource-group $RG_NAME --name $VM_NAME --yes --no-wait && \
az group delete --name $RG_NAME --yes --no-wait
```

# *Cronjob Commands*

*crontab files are stored in /var/spool/cron/crontabs*
*besides the default crontab file, users can create crontab files to schedule their own system events*

*edit the crontab file*
```bash
crontab -e
```

*add a cron job*
```bash
echo '* * * * * echo $(date) >> ~/logfile.txt' | crontab - && \
```
*`* * * * *`: runs the command every minute*
*`echo $(date) >> ~/logfile.txt`: writes the current time (date) to a file*

*verify cron job*
```bash
crontab -l
```

*clear all cron jobs*
```bash
crontab -r
```

*set and check current time zone*
```bash
sudo timedatectl set-timezone America/New_York
timedatectl
```

*run a python script every weekday at 09:00 AM*
```bash
echo '0 9 * * 1-5 /usr/bin/python3 /path/to/your_script.py' | crontab - && \
```
*`0 9 * * 1-5`: specifies the job should run at 09:00 AM every weekday*
*`/usr/bin/python3`: path to python interpreter*
*`/path/to/your_script.py`: path to python script*
