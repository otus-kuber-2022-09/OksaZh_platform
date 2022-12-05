Домашнее задание 5

Выполнение работы:

    Создан kind кластер.

    kind create cluster

    Создан MinIO на основе манифеста minio-statefulset.yaml:

    kubectl apply -f kubernetes-volumes/minio-statefulset.yaml

  В результате применения конфигурации должно произойти следующее:
      
      Запуститься под с MinIO
      Создаться PVC
      Динамически создаться PV на этом PVC с помощью дефолотного StorageClass

    Создан Headless Service на основе манифеста minio-headless-service.yamд MinIO изнутри кластера:

    kubectl apply -f kubernetes-volumes/minio-headless-service.yaml

  Проверка:

    kubectl get statefulsets

  NAME    READY   AGE
  minio   1/1     6m23s 

    kubectl get pods

  NAME      READY   STATUS    RESTARTS   AGE
  minio-0   1/1     Running   0          6m39s

    kubectl get pvc

     NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-bbeb20c5-ab8f-4f47-9888-69697bfd3177   10Gi       RWO            standard       13m

    kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-bbeb20c5-ab8f-4f47-9888-69697bfd3177   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                13m


    Задание со *:

    Согласно описанию secrets создан отдельный манифест с секратами - minio-secrets.yaml

    kubectl apply -f minio-secrets.yaml
    
   Секреты "зашифрованы" секреты в Base64
   
   Модифицирован манифест StatefulSet для использования volume типа secret - minio-statefulset_with_secret.yaml:

   kubectl apply -f minio-statefulset_with_secret.yaml
