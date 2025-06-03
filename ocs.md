# Architecture Concept - Red Hat OpenShift Container Storage (OCS)

* **GHE Issue:** [TBD](https://github.ibm.com)
* **Creation Date:**  Jun 30, 2020
* **Author:** Sandy Amin, Akash Gunjal

## Description of Problem
This document will discuss the OCS support to IBM Cloud RedHat OpenShift Kubernetes Service (ROKS).

## Solution Summary
Red Hat OpenShift Container Storage is software-defined storage that is optimised for container environments. It runs as an operator on OpenShift Container Platform to provide highly integrated and simplified persistent storage management for containers.

OCS will be provided a **priced add-on** for the ROKs Service.

### Architecture
![OpenShift Container Storage architecture](./images/ocs-architecture.png)

OpenShift Container Storage uses the following operators:

#### The OpenShift Container Storage (OCS) operator
This operator provides the storage cluster resource that wraps resources provided by the Rook-Ceph and NooBaa operators.

#### The Rook-Ceph operator
This operator automates the packaging, deployment, management, upgrading and scaling of persistent storage provided to container applications and infrastructure services provided to OpenShift Container Platform. It provides the Ceph cluster resource, which manages the pods that host services such as the Object Storage Daemons (OSDs), monitors and the metadata server for the Ceph file system.

#### The NooBaa operator
This operator provides the Multi-Cloud Object Gateway, an S3 compatible object store service that allows resource access across multiple cloud environments.

## High Level Design
- OCS is a Hyperconverged solution. There will be nodes which have storage drives used by OCS. If there is no storage on the node then it comes up as storage-less node.
- OCS in Independent mode: Planned for OCS4.6 version. This feature will enable OCS (via Rook) to manage a Ceph storage cluster that is external to the OpenShift cluster. The independent Ceph cluster can provide storage to many OpenShift clusters at the same time. The storage cluster can be a Ceph Cluster or an OCS dedicated cluster.
- As of OpenShift Container Storage 4.6, cluster reduction, whether by reducing OSDs or nodes, is not supported.

| **Features**         |**File (CephFS)** |**Block (Ceph RBD)** |**Object (NooBaa)** |
|----------------------|------------------|---------------------|--------------------|
|Dynamic Provisioning  |Yes               |Yes                  |Yes                 |
|Multizone-capable     |Yes               |Yes                  |Yes                 |
|Supported access + Volume mode |RWX + Filesystem |RWX + Block  |RWX + S3FS (COS-S3FS Plugin)|
|                               |RWO + Filesystem |RWO + Block  |RWO + S3FS (COS-S3FS Plugin)|
|                               |                 |RWO +  Filesystem |               |
|Performance           |High for all operations.  |High for all operations.  |TBD.   |
|Consistency           |Strong               |Strong            |Eventual            |
|Resiliency            |High as data is replicated across three zones.  |High as data is replicated across three zones.  |High  |
|Scalability           |Scales across multiple zones  |Scales across multiple zones  |High  |
|Encryption            |At rest  |At rest  |In transit and at rest  |

## Deployment
**Prerequisites:**
In order to deploy the OCS, at least one of these local storage options are required:
- Raw devices (no partitions or formatted filesystems)
- Raw partitions (no formatted filesystem)
- PVs available from a storage class in block mode

All worker nodes used for OCS deployment should have the label `cluster.ocs.openshift.io/openshift-storage=""`

Once the deployment is OCS done, you can verify the OCS is successful using the below command.
```
oc -n openshift-storage get cephclusters

Sample output:
NAME                             DATADIRHOSTPATH   MONCOUNT   AGE   PHASE   MESSAGE                        HEALTH
ocs-storagecluster-cephcluster   /var/lib/rook     3          21h   Ready   Cluster created successfully   HEALTH_OK
```
The default deployment will install the following operators and pods inside the cluster.
```
NAME                                                              READY   STATUS      RESTARTS   AGE   IP               NODE             NOMINATED NODE   READINESS GATES
csi-cephfsplugin-4gvv9                                            3/3     Running     0          24m   10.74.85.69      10.74.85.69      <none>           <none>
csi-cephfsplugin-5px5t                                            3/3     Running     0          24m   10.176.23.74     10.176.23.74     <none>           <none>
csi-cephfsplugin-bbf5n                                            3/3     Running     0          24m   10.176.23.123    10.176.23.123    <none>           <none>
csi-cephfsplugin-gkknl                                            3/3     Running     0          24m   10.209.153.96    10.209.153.96    <none>           <none>
csi-cephfsplugin-jx9pg                                            3/3     Running     0          24m   10.74.85.85      10.74.85.85      <none>           <none>
csi-cephfsplugin-ldkmq                                            3/3     Running     0          24m   10.209.153.79    10.209.153.79    <none>           <none>
csi-cephfsplugin-lpgst                                            3/3     Running     0          24m   10.74.85.116     10.74.85.116     <none>           <none>
csi-cephfsplugin-provisioner-8d5c6658f-dcwhl                      4/4     Running     0          35m   172.30.130.206   10.209.153.96    <none>           <none>
csi-cephfsplugin-provisioner-8d5c6658f-rjn28                      4/4     Running     0          35m   172.30.2.149     10.74.85.116     <none>           <none>
csi-cephfsplugin-ts5cd                                            3/3     Running     0          24m   10.209.153.107   10.209.153.107   <none>           <none>
csi-cephfsplugin-vc5b6                                            3/3     Running     0          24m   10.176.23.109    10.176.23.109    <none>           <none>
csi-rbdplugin-2fcj6                                               3/3     Running     0          24m   10.176.23.74     10.176.23.74     <none>           <none>
csi-rbdplugin-2jhwg                                               3/3     Running     0          25m   10.176.23.123    10.176.23.123    <none>           <none>
csi-rbdplugin-6vs7w                                               3/3     Running     0          25m   10.209.153.79    10.209.153.79    <none>           <none>
csi-rbdplugin-cpzlm                                               3/3     Running     0          25m   10.74.85.69      10.74.85.69      <none>           <none>
csi-rbdplugin-fdskb                                               3/3     Running     0          25m   10.176.23.109    10.176.23.109    <none>           <none>
csi-rbdplugin-hgmkj                                               3/3     Running     0          24m   10.209.153.107   10.209.153.107   <none>           <none>
csi-rbdplugin-kk8kv                                               3/3     Running     0          25m   10.209.153.96    10.209.153.96    <none>           <none>
csi-rbdplugin-provisioner-699f69fdb-55jl9                         4/4     Running     0          35m   172.30.217.193   10.176.23.123    <none>           <none>
csi-rbdplugin-provisioner-699f69fdb-qqn9w                         4/4     Running     0          35m   172.30.130.196   10.209.153.96    <none>           <none>
csi-rbdplugin-vxdtj                                               3/3     Running     0          25m   10.74.85.116     10.74.85.116     <none>           <none>
csi-rbdplugin-wg6z2                                               3/3     Running     0          25m   10.74.85.85      10.74.85.85      <none>           <none>
lib-bucket-provisioner-5ffc67fdd5-z4rjq                           1/1     Running     0          36m   172.30.44.71     10.74.85.85      <none>           <none>
noobaa-core-0                                                     2/2     Running     0          22m   172.30.217.200   10.176.23.123    <none>           <none>
noobaa-operator-595cc56f6b-wp9sh                                  1/1     Running     0          36m   172.30.180.70    10.176.23.109    <none>           <none>
ocs-operator-594d9c65-tmhhm                                       1/1     Running     0          36m   172.30.44.72     10.74.85.85      <none>           <none>
rook-ceph-drain-canary-10.176.23.123-84d6df5847-m5jzj             1/1     Running     0          28m   172.30.217.199   10.176.23.123    <none>           <none>
rook-ceph-drain-canary-10.209.153.96-7d564b89cc-rnw86             1/1     Running     0          28m   172.30.130.193   10.209.153.96    <none>           <none>
rook-ceph-drain-canary-10.74.85.116-59878f45c9-kr5ks              1/1     Running     0          28m   172.30.2.157     10.74.85.116     <none>           <none>
rook-ceph-mds-ocs-storagecluster-cephfilesystem-a-79d89b5bw2cwr   1/1     Running     0          28m   172.30.130.228   10.209.153.96    <none>           <none>
rook-ceph-mds-ocs-storagecluster-cephfilesystem-b-8b54bf8d84knh   1/1     Running     0          28m   172.30.2.156     10.74.85.116     <none>           <none>
rook-ceph-mgr-a-bbd4c59c-xjhtp                                    1/1     Running     0          29m   172.30.130.202   10.209.153.96    <none>           <none>
rook-ceph-mon-a-67888756bc-4fk4k                                  1/1     Running     0          30m   172.30.2.153     10.74.85.116     <none>           <none>
rook-ceph-mon-b-6c5957b6f6-hsg5n                                  1/1     Running     0          30m   172.30.130.203   10.209.153.96    <none>           <none>
rook-ceph-mon-c-65bbcf4ddf-xtcx4                                  1/1     Running     0          29m   172.30.217.250   10.176.23.123    <none>           <none>
rook-ceph-operator-5bdcccc49b-62rbz                               1/1     Running     0          36m   172.30.217.195   10.176.23.123    <none>           <none>
rook-ceph-osd-0-65647c4f57-zbb82                                  1/1     Running     0          28m   172.30.2.151     10.74.85.116     <none>           <none>
rook-ceph-osd-1-7f7c868bd5-bbqfv                                  1/1     Running     0          28m   172.30.217.198   10.176.23.123    <none>           <none>
rook-ceph-osd-2-5d994b46c8-q58wk                                  1/1     Running     0          28m   172.30.130.194   10.209.153.96    <none>           <none>
rook-ceph-osd-prepare-ocs-deviceset-0-0-h8p7q-lz7mg               0/1     Completed   0          29m   172.30.2.154     10.74.85.116     <none>           <none>
rook-ceph-osd-prepare-ocs-deviceset-1-0-vfvvb-jvnkh               0/1     Completed   0          29m   172.30.217.254   10.176.23.123    <none>           <none>
rook-ceph-osd-prepare-ocs-deviceset-2-0-cmgwx-z66xg               0/1     Completed   0          29m   172.30.130.195   10.209.153.96    <none>           <none>
rook-ceph-rgw-ocs-storagecluster-cephobjectstore-a-b8767fc9wkzt   1/1     Running     0          28m   172.30.2.159     10.74.85.116     <none>           <none>
```

**Check the health for NooBaa**

Run `kubectl get backingstore -n noobaa`, the backing store shold be in `Ready` phase
```
$ noobaa backingstore list
NAME                           TYPE      TARGET-BUCKET   PHASE      AGE        
ibm-cos-01                     ibm-cos   noobaastor01    Ready      3h37m44s    
noobaa-default-backing-store   pv-pool                   Ready      21h41m31s
```

There will be several entry points supported to deploy OCS for a ROKS environment.

**Phase 1 - ROKS Add-on**
The primary way that will be supported will be to offer OCS as an ROKS priced add-on. Users will be required to be provide input to deploy OCS across various ROKS and IBM Infrastructure types. We will have an `ibm-ocs-operator` which will be deployed through the `openshift-container-storage` add-on. The user can create custom resource `OcsCluster` of this operator to deploy OCS. The main job of the operator is to simplify the OCS deployment. It takes user inputs needed for OCS deployment and then create OCS resources such as OperatorGroup, Subscription and StorageCluster. This operator supports scale-up of OCS storage cluster to increase the total capacity of the storage. It also adds the required OCS labels on worker nodes which will be used for OCS by taking worker nodes as inputs in custom resource.

OCS resources created by ibm-ocs-operator:
- OperatorGroup
- Subscription
- StorageCluster

**Description of OcsCluster custom resource parameters**

| **Parameter Name** |**Description** |**Datatype** |**Mandatory** |**Default Value** |
|--------------------|----------------|-------------|--------------|------------------|
|monStorageClassName |Storage class for mon pods    |string |Yes   |                  |
|monSize             |Size for mon pods             |string |Yes   |                  |
|monDevicePaths      |Devices present on worker in case of local volumes |string array |No |                  |
|osdStorageClassName |Storage class for osd pods    |string |Yes   |                  |
|osdSize             |Size for mon pods             |string |Yes   |                  |
|osdDevicePaths      |Devices present on worker in case of local volumes |string array |No |                  |
|numOfOsd        |Number of OSDs required. OCS will create 3x number of OSDs for the value specified here |integer |No |1 |
|workerNodes         |Workers which need to be part of OCS |string array |No |        |
|billingType         |Billing option |enum [hourly/monthly] |No    |hourly            |
|ocsUpgrade          |Upgrade the OCS major version |boolean |No   |false            |

Sample Custom Resource for Classic SoftLayer cluster with all local storage
```
apiVersion: ocs.ibm.io/v1
kind: OcsCluster
metadata:
  name: ocscluster-classic
spec:
  monClass: localfile
  monSize: "1"
  monDevicePaths:
    - /dev/sdd
  osdClass: localblock
  osdSize: "1"
  osdDevicePaths:
    - /dev/sdc
  workerNodes:
    - 10.240.0.37
    - 10.240.0.38
    - 10.240.0.39
```

Sample Custom Resource for Classic SoftLayer cluster (**This is not recommended for production workloads currently**)
```
apiVersion: ocs.ibm.io/v1
kind: OcsCluster
metadata:
  name: ocscluster-classic
spec:
  monClass: ibmc-block-gold
  monSize: 20Gi
  osdClass: localblock
  osdSize: "1"
  osdDevicePaths:
    - /dev/sdc
  workerNodes:
    - 10.240.0.37
    - 10.240.0.38
    - 10.240.0.39
```

Sample Custom Resource for VPC Gen2 cluster
```
apiVersion: ocs.ibm.io/v1
kind: OcsCluster
metadata:
  name: ocscluster-vpc
spec:
  monClass: ibmc-vpc-block-10iops-tier
  monSize: 20Gi
  osdClass: ibmc-vpc-block-10iops-tier
  osdSize: 100Gi
  workerNodes:
    - 10.240.0.37
    - 10.240.0.38
    - 10.240.0.39
```

**Mapping of OCS deployment parameters passed by OcsCluster custom resource parameters**

| **OcsCluster Parameter** |**OCS Deployment Parameter** |
|--------------------------|-----------------------------|
|monStorageClassName       |monPVCTemplate.spec.storageClassName |
|monSize                   |monPVCTemplate.spec.resources.requests.storage |
|monDevicePaths            |LocalVolume.spec.storageClassDevices.devicePaths |
|osdStorageClassName       |storageDeviceSets.dataPVCTemplate.spec.storageClassName |
|osdSize                   |storageDeviceSets.dataPVCTemplate.spec.resources.requests.storage |
|osdDevicePaths            |LocalVolume.spec.storageClassDevices.devicePaths |
|numOfOsd                  |storageDeviceSets.count      |

**Phase 2 - As an integrated Service as part of the ROKS**
In this phase OCS will be offered as an integrated Service with all ROKS deployments.

## UX Experience
A new ROKS add-on will be used deploy a IBM operator. Enable add-on will report failures if it fails to deploy the operator.
### Enable add-on
`ibmcloud ks cluster addon enable openshift-container-storage --cluster <CLUSTER_NAME>`

### List deployed add-ons
`ibmcloud ks cluster addon ls --cluster <CLUSTER_NAME>`

### Disable add-on
`ibmcloud ks cluster addon disable openshift-container-storage --cluster <CLUSTER_NAME>`

## Billing
OCS will be offered as a priced add-on under the ROKS and Satellite offerings. A new part number will be used for billing purposes of OCS. Then general billing design will be to track usage metrics based on the number of 2-core pairs utilised by the OCS nodes use to provide storage within the customer cluster.

This concept document has the details of the [Billing Design](https://github.ibm.com/alchemy-containers/armada/blob/storage-billing/architecture/guild/concept-docs/concept-openshift-ocs-billing-metering.md)

## Capacity Expansion of OCS (Scaling)
OCS storage capacity can be expanded by the below ways.

### Adding additional worker nodes to the cluster
Worker nodes can be added either through ibm-ocs-operator or using RedHat OpenShift console
- Add worker node through IBM Cloud using ibm-ocs-operator
  1. Expand the worker pool of the cluster used for OCS by adding more worker nodes.
  2. If its a bare metal, then the storage capacity will be increased by additional SSD disks. If its a VSI (Supported only on VPC clusters), then ensure to create a remote block volume using `ibmcloud is` command and attach it on the worker using https://cloud.ibm.com/docs/containers?topic=containers-cli-plugin-kubernetes-service-cli#cs_storage_att_cr
  3. Add the label `cluster.ocs.openshift.io/openshift-storage=""` on the new worker nodes. Or if the list of worker nodes have been provided in `OcsCluster` resource during OCS deploy, then add edit the `OcsCluster` and add this worker node.
  4. Confirm that at least the following pods on the new node are in Running state: `csi-cephfsplugin-*`, `csi-rbdplugin-*`

- Add worker node through RedHat OpenShift console
  1. Refer https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.5/html/managing_openshift_container_storage/scaling-storage-nodes_rhocs#scaling-out-storage-capacity_rhocs to add worker node using RedHat OpenShift web console.

### Adding additional storage disks to the worker node
Storage disks can be added either through ibm-ocs-operator or using RedHat OpenShift console
- Add storage disks through IBM Cloud using ibm-ocs-operator
  1. If you are using local storage (SSD disks), then ensure you have sufficient additional disk on the worker node and create a local volume for that disk. No need to create volumes if remote storage is used.
  2. Edit the `OcsCluster` resource and update the `numOfOsd` value to the number of OSDs required. OCS will create 3x number of OSDs for the value specified here. So the OCS will add new disks and increase the capacity.

- Add storage disks through RedHat OpenShift console
  1. Refer https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.5/html/managing_openshift_container_storage/scaling-storage-nodes_rhocs#scaling-up-storage-capacity_rhocs to add storage disks to the worker node using RedHat OpenShift web console.

## Upgrade
- We install OCS based on channels given by RedHat like stable-4.6 and whenever there are updates to existing version through channel then the OCS will get upgraded automatically.
- Upgrade the major version of OCS (like OCS 4.5 to OCS 4.6) with either of the below options.
  - Upgrade through IBM Cloud using ibm-ocs-operator
    1. Edit the `OcsCluster` resource and update the `ocsUpgrade` value to `true` so the `ibm-ocs-operator` will update the OCS version through modifying the channel in the subscription.

  - Upgrade through RedHat OpenShift console
    1. Refer https://access.redhat.com/documentation/en-us/red_hat_openshift_container_storage/4.5/html/managing_openshift_container_storage/upgrading-your-cluster_rhocs to upgrade using RedHat OpenShift web console.

## Add-on health integration
We will add kube health checks in OCS add-on to determine the health of the add-on. This health check will monitor the status of Ceph Cluster and once its in ready state, it reports add-on as healthy.

## Logging and Monitoring
RedHat OpenShift web console has a dashboard which can be used for monitoring and checking various metrics and alerts including OCS.
Refer https://cloud.ibm.com/docs/openshift?topic=openshift-health#oc_logmet_options for various logging and monitoring options available with RedHat OpenShift on IBM Cloud.

## Auditing
There are events generated on RedHat OpenShift web console dashboard which can be checked for recent activities on the cluster.

## Issues or Concerns
- We have some open issues being tracked with RedHat development team and most of those issues will be addressed as part of OCS 4.6

## Troubleshooting
### What to do when OCS install fails
- Check Armada-Health status by looking at YYY
- If the Ceph cluster is not healthy then check the describe of ceph cluster as using `oc -n openshift-storage describe cephcluster <Ceph_Cluster_Name>`
- You can use the toolbox available in rook community to debug OCS deploy issues. Refer https://rook.io/docs/rook/v1.3/ceph-toolbox.html to use toolbox.
- Use command `oc debug node/<Worker_Node>` and then run the command `lsblk` to check if your disks used in OCS are raw devices or raw partitions and they do not have any filesystems on them.
```
Sample output:
lsblk -f
NAME                  FSTYPE      LABEL UUID                                   MOUNTPOINT
vda
└─vda1                LVM2_member       eSO50t-GkUV-YKTH-WsGq-hNJY-eKNf-3i07IB
  ├─ubuntu--vg-root   ext4              c2366f76-6e21-4f10-a8f3-6776212e2fe4   /
  └─ubuntu--vg-swap_1 swap              9492a3dc-ad75-47cd-9596-678e8cf17ff9   [SWAP]
vdb
```

### How to cleanup OCS as part of un-Install
- After disabling the add-on, the OCS storage cluster and operator will get deleted.
- Ensure the local disks used in OCS are raw without any filesystems.
