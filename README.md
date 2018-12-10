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