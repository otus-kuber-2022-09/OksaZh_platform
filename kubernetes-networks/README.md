Домашнее задание 3

- Выполнена установка кластера kind
- Применена ReplicaSet для сервиса frontend:
  kubectl apply -f frontend-replicaset.yaml

- Вопрос: Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod?

ReplicaSet controller не рестартует поды, а следит за тем, чтобы объявленное количество подов было запущено в каждый момент времени

 - Создан манифест paymentservice-deployment.yaml. Выполнено обновление и роллбек версий приложения, используя deployment:
Версия 0.00.1:
kubectl apply -f paymentservice-deployment.yaml
kubectl get pods -l app=paymentservice -w
kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'

Версия 0.00.2:
kubectl apply -f paymentservice-deployment.yaml
kubectl get pods -l app=paymentservice -w
kubectl get pods -l app=paymentservice -o=jsonpath='{.items[0:3].spec.containers[0].image}'

Rollback на v0.0.1:

rollout kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w

- Созданы манифесты для  Blue/Green и Reverse Rolling Update с использованием параметров maxSurge и maxUnavailable

Blue/Green:

kubectl apply -f paymentservice-deployment-bg.yaml
kubectl get pods -l app=frontend -w

Reverse Rolling Update:

kubectl apply -f paymentservice-deployment-reverse.yaml
kubectl get pods -l app=frontend -w

- Выполнена работа с настройками readinessProbe и livenessProbe
- Выполнена установка node-exporter на master и worker nodes используя DaemonSet

kubectl apply -f node-exporter-daemonset.yaml
kubectl port-forward node-exporter-mzcfw 9100:9100
curl localhost:9100/metrics

- Для развертывания Node Exporter на master используется tolerations 

      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
