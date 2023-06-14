
### Build AKS Clusters (optional)
``` 
az aks create \
        --resource-group ${AZ_RESOURCE_GROUP} \
        --name ${AZ_CLUSTER_NAME} \
        --generate-ssh-keys \
        --vm-set-type VirtualMachineScaleSets \
        --node-vm-size ${AZ_VM_SIZE} \
        --load-balancer-sku standard \
        --enable-managed-identity \
        --node-count 2 \
        --zones 1 
```
## Installing Velero Client
This must be installed in the machine you are issuing the velero command from

https://velero.io/docs/v1.8/basic-install/

## Installing Minio

Refer to this page for how to install Minio: https://min.io/

## Configuring Minio for Velero 
For official Azure documentation on Velero for AKS, refer to: https://learn.microsoft.com/en-us/azure/aks/hybrid/backup-workload-cluster#install-velero-with-minio-storage

1. Create a MinIO credentials file with the following information:

minio.credentials 
```
[default] 
  aws_access_key_id=<minio_access_key> 
  aws_secret_access_key=<minio_secret_key>
```


## Installing Velero Controller

This controller will create a new namespace called "velero" within your AKS cluster. Also notice that I am using the aws plugin since my Velero bucket will be Minio. If you are planning to use Blob storage, refer to the links above for the proper command 


``` 
velero install --provider aws --bucket velero-backup --secret-file $VELERO_CREDENTIALS --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=[YourBucketforVelero] --plugins velero/velero-plugin-for-aws:v1.7.0
```

Example
```
 velero install --provider aws --bucket velero-backup --secret-file $VELERO_CREDENTIALS --backup-location-config region=minio,s3ForcePathStyle=true,s3Url=http://20.229.42.74:9000 --plugins velero/velero-plugin-for-aws:v1.7.0
```

## Mx4PC - Scale down deployments
```
kubectl scale deployment mendix-operator --replicas=0
deployment.apps/mendix-operator scaled
```

```
kubectl scale deployment mendix-agent --replicas=0
deployment.apps/mendix-agent scaled
```
### Creating a new Velero backup

```
velero create backup aks-workshop-june
``` 

### Restoring from Backup
```
velero restore create --from-backup [nameofbackup] --status-include-resources=storageinstances.privatecloud.mendix.com,storageplans.privatecloud.mendix.com,builds.privatecloud.mendix.com,mendixapps.privatecloud.mendix.com
```
```
Example:
velero restore create --from-backup aks-velero-june --status-include-resources=storageinstances.privatecloud.mendix.com,storageplans.privatecloud.mendix.com,builds.privatecloud.mendix.com,mendixapps.privatecloud.mendix.com
```

## Patching StorageInstance

After completing the restore, add finalizers to StorageInstances (this step is optional, but highly recommended to ensure that Kubernetes garbage collection will clean up storage from deleted environments)

```
kubectl patch storageinstances $(kubectl get storageinstances --no-headers -o custom-columns=":metadata.name") -p '{"metadata":{"finalizers":["finalizer.privatecloud.mendix.com"]}}' --type=merge
```
