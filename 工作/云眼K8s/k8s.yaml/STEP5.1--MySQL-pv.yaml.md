```yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  namespace: yucekj
  labels:
    pv: pv-test
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    path: /root/nfs_root
    server: 10.0.10.120


```