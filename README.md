# gke-job-to-disk

Write to pvc.

```shell
kubectl apply -f ssd-storageclass.yaml

kubectl exec $(kubectl get pods --selector=job-name=write-to-disk --output=jsonpath={.items..metadata.name})  -c count -- tail /var/log/1.log
```

Once tail is confirmed. Kill Job

```shell
kubectl delete jobs/write-to-disk
```

Backup PV

```shell
DISK=$(kubectl get pv $(kubectl get pvc/ssd-data --output=jsonpath={.spec.volumeName}) --output=jsonpath='{.spec.gcePersistentDisk.pdName}{"\t --zone="}{.metadata.labels.failure\-domain\.beta\.kubernetes\.io/zone}')
```

```shell
gcloud compute disks describe $DISK
gcloud compute disks snapshot $DISK 
DISK_LINK=`gcloud compute disks describe $DISK --format="value(selfLink)"`
ZONE=`echo $DISK | awk -F'\=' '{print $2}'`
DISK=`gcloud compute disks describe $DISK --format="value(name)"`
```

Retrieve PV

```shell
FIRST_SNAPSHOT=`gcloud compute snapshots list --filter="(sourceDisk=$DISK_LINK)" --format="csv[no-heading,delimiter=','](name)" | head -n 1`
```

Create Disk from that snapshot

```shell
gcloud compute disks create restored-pd-ssd --size=30GB --source-snapshot=$FIRST_SNAPSHOT --type=pd-ssd --zone=$ZONE
```

Create pod to read from snapshot

```shell
kubectl apply -f pod-read-ssd.yaml
```