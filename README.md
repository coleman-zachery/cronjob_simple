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

# Cronjob Commands

*crontab files are stored in /var/spool/cron/crontabs*
*besides the default crontab file, users can create crontab files to schedule their own system events*

*edit the crontab file*
```bash
crontab -e
```

*add a cron job*
```bash
echo '* * * * * echo $(date) >> ~/logfile.txt' | crontab -
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
echo '0 9 * * 1-5 /usr/bin/python3 /path/to/your_script.py >/dev/null 2>&1' | crontab -
```
*`0 9 * * 1-5`: specifies the job should run at 09:00 AM every weekday*
*`/usr/bin/python3`: path to python interpreter*
*`/path/to/your_script.py`: path to python script*
*`>/dev/null 2>&1`: avoids Mail Transfer Agent (MTA) script resolution issues by suppressing standard error output*

*check error logs*
```bash
grep CRON /var/log/syslog
```

# Cron Job Time Frequency Parameters

Cron jobs are used to schedule tasks on Unix-based systems. This tutorial covers the syntax for defining time frequency parameters in a crontab file.

## Crontab Syntax

A cron job entry in the crontab file follows this format:

```
* * * * * /path/to/command
- - - - -
| | | | |
| | | | +----- Day of the Week (0 - 7) (Sunday is both 0 and 7)
| | | +------- Month (1 - 12)
| | +--------- Day of the Month (1 - 31)
| +----------- Hour (0 - 23)
+------------- Minute (0 - 59)
```


## Time Frequency Parameters

1. **Minute** (`0-59`)

   Specifies the minute of the hour when the command should run.
   - `0`: At the start of the hour.
   - `15`: At 15 minutes past the hour.
   - `*/5`: Every 5 minutes.

2. **Hour** (`0-23`)

   Specifies the hour of the day when the command should run.
   - `0`: At midnight.
   - `8`: At 8 AM.
   - `*/2`: Every 2 hours.

3. **Day of the Month** (`1-31`)

   Specifies the day of the month when the command should run.
   - `1`: On the 1st of the month.
   - `15`: On the 15th of the month.
   - `*/5`: Every 5 days.

4. **Month** (`1-12`)

   Specifies the month when the command should run.
   - `1`: In January.
   - `6`: In June.
   - `*/3`: Every 3 months.

5. **Day of the Week** (`0-7`)

   Specifies the day of the week when the command should run. Both `0` and `7` represent Sunday.
   - `0`: On Sunday.
   - `2`: On Tuesday.
   - `*/2`: Every 2 days (e.g., Sunday, Tuesday, Thursday, Saturday).

## Examples

1. **Run a Command Every Minute**
   ```cron
   * * * * * /path/to/command
   ```

2. **Run a Command Every Day at Midnight**
   ```cron
   0 0 * * * /path/to/command
   ```

3. **Run a Command Every Hour on the 15th of Each Month**
   ```cron
   0 * 15 * * /path/to/command
   ```

4. **Run a Command at 5 AM Every Monday**
   ```cron
   0 5 * * 1 /path/to/command
   ```

5. **Run a Command Every 5 Minutes**
   ```cron
   */5 * * * * /path/to/command
   ```

6. **Run a Command at 2 PM on Weekdays**
   ```cron
   0 14 * * 1-5 /path/to/command
   ```

7. **Run a Command on the 1st of Every Month at 12:30 PM**
   ```cron
   30 12 1 * * /path/to/command
   ```

## Special Characters
- `*` **(Asterisk)**: Represents every value in that field.
- `,` **(Comma)**: Specifies additional values (e.g., 1,15).
- `-` **(Dash)**: Defines a range of values (e.g., 1-5).
- `/` **(Slash)**: Specifies intervals (e.g., */15).
