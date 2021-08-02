

https://aws.amazon.com/ko/blogs/storage/persistent-storage-for-container-logging-using-fluent-bit-and-amazon-efs/

kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-
driver/deploy/kubernetes/overlays/stable?ref=master"


cat storageclass.yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com

curl -o pv.yaml  https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/examples/kubernetes/multiple_pods/specs/pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-56ee5a53::fsap-08e12b7694ed54eb9 