# This is for backup and retore: iteration over managedcluster to get only the one with then name
- name: If the work order printing module is listed in inventory then execute its tasks
  include: woprinting.yml
  when: "'WorkOrderPrinting' in (web_module_list | map(attribute='module_name') )"

...
with_items:
  - 1
  - 2
  - 3
when: (item != 2) or (item == 2 and test_var is defined)

## Vedere se si puo' fare

  - with_items:
    - "{{ managedcluster_list.resources }}"
    when: ( item.name != 'local-cluster' ) and( item.name in ( aks_list | map(attribue='name') )


# This is for the backup and restore


az storage account list --subscription c7130a18-cbc9-4b44-9307-1f56446fb730 --query [].name -o tsv


```shell
$ export RESOURCEGROUP=sdminone-aoc12-backup-rg
$ export SUBSCRIPTION=c7130a18-cbc9-4b44-9307-1f56446fb730
$ export STORAGEACCOUNT=sdminonnebackupsa
$ export CONTAINER=aoc12-backup-restore-test
```


It creates resource group

```shell
$ az group create -l eastus --name ${RESOURCEGROUP} --subscription ${SUBSCRIPTION}
```

It creates the storage account and retrieve the storage key

```shell
$ az storage account create --name ${STORAGEACCOUNT} -l eastus --sku Standard_LRS -g ${RESOURCEGROUP} --subscription ${SUBSCRIPTION}
$ STORAGEKEY=$(az storage account keys list --account-name ${STORAGEACCOUNT} -g ${RESOURCEGROUP} --subscription ${SUBSCRIPTION} --query [0].value -o tsv)
```

it puts storage key in `storage.secret` file

```shell
$ cat << EOF > operators/cluster-backup/enable-cluster-backup/overlays/azure/storage.secret
AZURE_STORAGE_ACCOUNT_ACCESS_KEY=${STORAGEKEY}
AZURE_CLOUD_NAME=AzurePublicCloud
EOF
```
It crates the container aka the Azure equivalent of the AWS S3 bucket.

```shell
$ az storage container create --account-name ${STORAGEACCOUNT} -n ${CONTAINER} -g  ${RESOURCEGROUP} --subscription ${SUBSCRIPTION}
```



It configures the `Velero` CRD with all the needed info


```shell
$ cat << EOF > operators/cluster-backup/enable-cluster-backup/overlays/azure/velero.yaml
apiVersion: oadp.openshift.io/v1alpha1
kind: Velero
metadata:
  name: velero
  namespace: openshift-adp
spec:
  enableRestic: false
  defaultVeleroPlugins:
  - azure
  - openshift
  backupStorageLocations:
  - name: default
    default: true
    provider: azure
    credential:
      key: cloud
      name: cloud-credentials-azure
    objectStorage:
      bucket: ${CONTAINER}
      prefix: backups
    config:
      resourceGroup: ${RESOURCEGROUP}
      storageAccount: ${STORAGEACCOUNT}
      subscriptionId: ${SUBSCRIPTION}
      storageAccountKeyEnvVar: AZURE_STORAGE_ACCOUNT_ACCESS_KEY
EOF
```
