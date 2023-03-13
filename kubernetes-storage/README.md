Домашнее задание 13

1. Запуск minikube:
minikube start

2. Настройка external-snapshotter:

git clone https://github.com/kubernetes-csi/external-snapshotter.git
cd external-snapshotter
kubectl create -f client/config/crd

customresourcedefinition.apiextensions.k8s.io/volumesnapshotclasses.snapshot.storage.k8s.io created
customresourcedefinition.apiextensions.k8s.io/volumesnapshotcontents.snapshot.storage.k8s.io created
customresourcedefinition.apiextensions.k8s.io/volumesnapshots.snapshot.storage.k8s.io created

Установка СSI:

 git clone https://github.com/kubernetes-csi/csi-driver-host-path.git
 cd csi-driver-host-path/deploy/kubernetes-1.24/
./deploy.sh


kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
csi-hostpath-socat-0   1/1     Running   0          44s
csi-hostpathplugin-0   8/8     Running   0          48s
csi-snapshotter-0      3/3     Running   0          9m47s

Применение yaml из example:

for i in ./examples/csi-storageclass.yaml ./examples/csi-pvc.yaml ./examples/csi-app.yaml; do kubectl apply -f $i; done
storageclass.storage.k8s.io/csi-hostpath-sc configured
persistentvolumeclaim/csi-pvc created
pod/my-csi-app created

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS      REASON   AGE
pvc-db34579c-0cf2-49cb-8c5d-72b01a42defc   1Gi        RWO            Delete           Bound    default/csi-pvc   csi-hostpath-sc            2m18s

kubectl get pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc   Bound    pvc-db34579c-0cf2-49cb-8c5d-72b01a42defc   1Gi        RWO            csi-hostpath-sc   3m7s

kubectl describe pods/my-csi-app
Name:             my-csi-app
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/172.23.189.203
Start Time:       Mon, 03 Mar 2023 12:04:38 +0300
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.10
IPs:
  IP:  172.17.0.10
Containers:
  my-frontend:
    Container ID:  docker://6a14b8f0bbbf4476d8652774ecafe63026345a1462eb7b25d6ad454c93a826e5
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:7b3ccabffc97de872a30dfd234fd972a66d247c8cfc69b0550f276481852627c
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      1000000
    State:          Running
      Started:      Mon, 03 Mar 2023 12:04:49 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from my-csi-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-94mwn (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  my-csi-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  csi-pvc
    ReadOnly:   false
  kube-api-access-94mwn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                   Age                     From                                      Message
  ----     ------                   ----                    ----                                      -------
  Warning  FailedScheduling         3m40s                   default-scheduler                         0/1 nodes are available: 1 persistentvolumeclaim "csi-pvc" not found. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Warning  FailedScheduling         3m25s                   default-scheduler                         0/1 nodes are available: 1 pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Scheduled                3m22s                   default-scheduler                         Successfully assigned default/my-csi-app to minikube
  Warning  VolumeConditionAbnormal  3m22s (x10 over 3m22s)  csi-pv-monitor-agent-hostpath.csi.k8s.io  The volume isn't mounted
  Normal   SuccessfulAttachVolume   3m22s                   attachdetach-controller                   AttachVolume.Attach succeeded for volume "pvc-db34579c-0cf2-49cb-8c5d-72b01a42defc"
  Normal   Pulling                  3m16s                   kubelet                                   Pulling image "busybox"
  Normal   Pulled                   3m11s                   kubelet                                   Successfully pulled image "busybox" in 5.247597056s
  Normal   Created                  3m11s                   kubelet                                   Created container my-frontend
  Normal   Started                  3m11s                   kubelet                                   Started container my-frontend
  Normal   VolumeConditionNormal    82s (x15 over 2m22s)    csi-pv-monitor-agent-hostpath.csi.k8s.io  The Volume returns to the healthy state


Проверка секций:

Containers.my-frontend.Mounts - в контейнер замонтирован volume my-csi-volume в директорию /data.
Volunes - my-csi-volume это персистентное хранилице мозданое по заявке csi-pvc
Events - в ивентах можно увидеть, что volume успешно приатачен.


Проверка работы HostPath driver:

Создан файл в /data директории внутри конейнера в поде my-csi-app:

kubectl exec -it my-csi-app -- /bin/sh

touch /data/hello-world
ls -la /data
total 4
drwxr-xr-x    2 root     root            60 Mar 3 12:33 .
drwxr-xr-x    1 root     root          4096 Mar 3 12:41 ..
-rw-r--r--    1 root     root             0 Mar 3 12:33 hello-world
exit

Проверка, что файл появился в HostPath контейнере:

minikube ssh

sudo find / -name hello-world
/var/lib/csi-hostpath-data/4bf4d2b9-7b63-11ea-81fd-0242ac110008/hello-world
/var/lib/kubelet/pods/2f145fe2-6c87-451f-a105-9d1911b21adf/volumes/kubernetes.io~csi/pvc-829fad3b-ef60-4640-865f-dd02dbbaffda/mount/hello-world
/mnt/sda1/var/lib/kubelet/pods/2f145fe2-6c87-451f-a105-9d1911b21adf/volumes/kubernetes.io~csi/pvc-829fad3b-ef60-4640-865f-dd02dbbaffda/mount/hello-world

3. Выполнение ДЗ

Создать StorageClass для CSI Host Path Driver
На своей тестовой машине его нужно установить самостоятельно
Создать объект PVC c именем storage-pvc
Создать объект Pod c именем storage-pod
Хранилище нужно смонтировать в /data

cd kubernetes-storage

Все манифесты для создания находятся в директориии hw

for i in ./hw/storageClass.yaml ./hw/pvc.yaml ./hw/pod.yaml; do kubectl apply -f $i; done

Проверка

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS      REASON   AGE
pvc-91ee963f-2fc3-42ec-a428-017acf8969ab   1Gi        RWO            Delete           Bound    default/storage-pvc   otus-sc                    21s
pvc-db34579c-0cf2-49cb-8c5d-72b01a42defc   1Gi        RWO            Delete           Bound    default/csi-pvc       csi-hostpath-sc            37m

kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc       Bound    pvc-db34579c-0cf2-49cb-8c5d-72b01a42defc   1Gi        RWO            csi-hostpath-sc   38m
storage-pvc   Bound    pvc-91ee963f-2fc3-42ec-a428-017acf8969ab   1Gi        RWO            otus-sc           53s



kubectl describe pods/storage-pod
Name:             storage-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/172.23.189.203
Start Time:       Mon, 03 Mar 2023 12:42:10 +0300
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               172.17.0.11
IPs:
  IP:  172.17.0.11
Containers:
  app:
    Container ID:  docker://fb2477b5f2adaf6efa36afe224f171217cf7253b55682600029d1915dfe087dc
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:7b3ccabffc97de872a30dfd234fd972a66d247c8cfc69b0550f276481852627c
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      1000000
    State:          Running
      Started:      Mon, 03 Mar 2023 12:42:16 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from data-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vjn82 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  data-volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  storage-pvc
    ReadOnly:   false
  kube-api-access-vjn82:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age                From                     Message
  ----     ------                  ----               ----                     -------
  Warning  FailedScheduling        55s (x2 over 61s)  default-scheduler        0/1 nodes are available: 1 pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Scheduled               52s                default-scheduler        Successfully assigned default/storage-pod to minikube
  Normal   SuccessfulAttachVolume  51s                attachdetach-controller  AttachVolume.Attach succeeded for volume "pvc-91ee963f-2fc3-42ec-a428-017acf8969ab"
  Normal   Pulling                 48s                kubelet                  Pulling image "busybox"
  Normal   Pulled                  46s                kubelet                  Successfully pulled image "busybox" in 2.054096024s
  Normal   Created                 46s                kubelet                  Created container app
  Normal   Started                 46s                kubelet                  Started container app


  4. Тест snapshot:

Добавлен VolumeSnapshotClass

kubectl apply -f volumeSnapshotClass.yaml
volumesnapshotclass.snapshot.storage.k8s.io/test-snapclass created

Даные для snapshot:

kubectl exec -it storage-pod -- /bin/sh/

touch /data/hello-world
touch /data/test
touch /data/test2
exit

Создание snapshot:

kubectl apply -f snapshotVolume.yaml


kubectl describe volumesnapshot
Name:         test-snapshot
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  snapshot.storage.k8s.io/v1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2023-03-03T09:50:35Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:  1
  Managed Fields:
    API Version:  snapshot.storage.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:source:
          .:
          f:persistentVolumeClaimName:
        f:volumeSnapshotClassName:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2023-03-03T09:50:35Z
    API Version:  snapshot.storage.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          v:"snapshot.storage.kubernetes.io/volumesnapshot-bound-protection":
      f:status:
        .:
        f:boundVolumeSnapshotContentName:
    Manager:      snapshot-controller
    Operation:    Update
    Time:         2023-03-03T09:50:35Z
    API Version:  snapshot.storage.k8s.io/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:creationTime:
        f:readyToUse:
        f:restoreSize:
    Manager:         snapshot-controller
    Operation:       Update
    Subresource:     status
    Time:            2023-03-03T09:50:36Z
  Resource Version:  69673
  UID:               cdc29df5-d9bc-48fd-af0e-af282ea022ae
Spec:
  Source:
    Persistent Volume Claim Name:  storage-pvc
  Volume Snapshot Class Name:      test-snapclass
Status:
  Bound Volume Snapshot Content Name:  snapcontent-cdc29df5-d9bc-48fd-af0e-af282ea022ae
  Creation Time:                       2023-03-03T09:50:35Z
  Ready To Use:                        true
  Restore Size:                        1Gi


Добавление данных в контейнере, уже после создания snapshot'a, чтобы позже сравнить:

kubectl exec -it storage-pod -- /bin/sh

ls -na /data
total 4
drwxr-xr-x    2 0        0              100 Mar 3 12:33 .
drwxr-xr-x    1 0        0             4096 Mar 3 12:41 ..
-rw-r--r--    1 0        0                0 Mar 3 12:33 hello-world
-rw-r--r--    1 0        0                0 Mar 3 12:33 test
-rw-r--r--    1 0        0                0 Mar 3 12:33 test2
touch /data/after-snapshot
ls -na /data
total 4
drwxr-xr-x    2 0        0              120 Mar 3 13:38 .
drwxr-xr-x    1 0        0             4096 Mar 3 13:40 ..
-rw-r--r--    1 0        0                0 Mar 3 13:38 after-snapshot
-rw-r--r--    1 0        0                0 Mar 3 13:25 hello-world
-rw-r--r--    1 0        0                0 Mar 3 13:25 test
-rw-r--r--    1 0        0                0 Mar 3 13:25 test2
exit

PVC из снапшота:

kubectl apply -f pvcFromSnapshot.yaml
persistentvolumeclaim/storage-pvc-resored created

Проверка нового persdidtentVolume:

kubectl get pvc
NAME                  STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
csi-pvc               Bound     pvc-db34579c-0cf2-49cb-8c5d-72b01a42defc   1Gi        RWO            csi-hostpath-sc   56m
storage-pvc           Bound     pvc-91ee963f-2fc3-42ec-a428-017acf8969ab   1Gi        RWO            otus-sc           19m
storage-pvc-resored   Bound                                                                            otus-sc           40s


kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS      REASON   AGE
pvc-3393f4eb-edb6-4284-af14-0625b61ea040   1Gi        RWO            Delete           Bound    default/storage-pvc           otus-sc                    63m
pvc-829fad3b-ef60-4640-865f-dd02dbbaffda   1Gi        RWO            Delete           Bound    default/csi-pvc               csi-hostpath-sc            122m
pvc-e12f6140-3ad2-41dc-820d-7b92335c72a4   1Gi        RWO            Delete           Bound    default/storage-pvc-resored   otus-sc


Создание pod c востановленым из снапшота volume:

kubectl apply -f podFromSnapshot.yaml
pod/storage-pod-restored created

Проверка что внутри:

kubectl exec -it storage-pod-restored -- /bin/sh


ls -na /data
total 4
drwxr-xr-x    2 0        0              100 Mar 3 12:33 .
drwxr-xr-x    1 0        0             4096 Mar 3 12:41 ..
-rw-r--r--    1 0        0                0 Mar 3 12:33 hello-world
-rw-r--r--    1 0        0                0 Mar 3 12:33 test
-rw-r--r--    1 0        0                0 Mar 3 12:33 test2

