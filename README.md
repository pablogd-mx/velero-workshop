### Build Clusters
You can refer to any of our quickstarts:
For Azure: https://github.com/pablogd-mx/azure-quickstart

For AWS: https://github.com/ssahadevan-mendix/aws-mendix-quickstart

For GCP: https://github.com/pablogd-mx/gcp-quickstart


### Installing Velero Client
https://velero.io/docs/v1.8/basic-install/

### Velero driver for EKS

https://github.com/vmware-tanzu/velero-plugin-for-aws


## Creating a new S3 Bucket to Store Velero Backups.
```
BUCKET=<YOUR_BUCKET>
REGION=<YOUR_REGION>
aws s3api create-bucket \
    --bucket $BUCKET \
    --region $REGION \
    --create-bucket-configuration LocationConstraint=$REGION
```
```
BUCKET=veleros3dr
REGION=eu-west-2
aws s3api create-bucket \
    --bucket $BUCKET \
    --region $REGION \
    --create-bucket-configuration LocationConstraint=$REGION
```
## Create IAM user ( optional but highly recommended)
```
aws iam create-user --user-name velero
```

## Attach Policy to velero user to access S3 bucket ( see policy example in link above)

```
aws iam put-user-policy \
  --user-name velero \
  --policy-name velero \
  --policy-document file://velero-policy.json
```

## Create an access key for the user
aws iam create-access-key --user-name velero

## Installing Velero into K8s cluster.
### Make sure it's version 1.5.0 or higher, otherwise it won't work with Mx4PC.
```
velero install \
    --provider aws \
    --plugins velero/velero-plugin-for-aws:v1.5.0 \
    --bucket $BUCKET \
    --backup-location-config region=$REGION \
    --snapshot-location-config region=$REGION \
    --secret-file ./credentials-velero
```
## Create a new backup location ( optional but highly recommended)
### First create a secret with Velero Credentials obtained in previous step
```
kubectl create secret generic -n velero [nameofyoursecret] --from-file=aws=credentials-velero

Example:
kubectl create secret generic -n velero bsl-credentials --from-file=aws=credentials-velero
```
### Create a new backup location
```
velero backup create [nameofbackup] --storage-location [nameofstoragelocation]

velero backup create mendix-full --storage-location bsl-mendix
```

### Restoring from Backup
```
velero restore create --from-backup [nameofbackup] --status-include-resources=storageinstances.privatecloud.mendix.com,storageplans.privatecloud.mendix.com,builds.privatecloud.mendix.com,mendixapps.privatecloud.mendix.com
```
```
Example:
velero restore create --from-backup demo-mxdr --status-include-resources=storageinstances.privatecloud.mendix.com,storageplans.privatecloud.mendix.com,builds.privatecloud.mendix.com,mendixapps.privatecloud.mendix.com
```