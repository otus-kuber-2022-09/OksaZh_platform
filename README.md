Задание 2.1:
(Разберитесь почему все pod в namespace kube-system восстановились после удаления.):

pod-ы кроме coredns - static и управляются напрямую kubelet'ом: $ ls -lht /etc/kubernetes/manifests/

-rw------- 1 root root 2.4K Oct 3 19:31 etcd.yaml
-rw------- 1 root root 3.6K Oct 3 19:31 kube-apiserver.yaml
-rw------- 1 root root 2.9K Oct 3 19:31 kube-controller-manager.yaml
-rw------- 1 root root 1.5K Oct 3 19:31 kube-scheduler.yaml

coredns описан в деплойменте: kubectl describe deployments coredns -n kube-system

Name: coredns
Namespace: kube-system
CreationTimestamp: Mon, 03 Oct 2022 22:31:25 +0300
Labels: k8s-app=kube-dns
Annotations: deployment.kubernetes.io/revision: 1
Selector: k8s-app=kube-dns
Replicas: 1 desired | 1 updated | 1 total | 1 available | 0 unavailable

Pod будет восстановлено автоматически.

Задание 2.2.

Для выполнения домашней работы необходимо создать Dockerfile, в
котором будет описан образ:
Запускающий web-сервер на порту 8000 (можно использовать любой
способ);
Отдающий содержимое директории /app внутри контейнера (например,
если в директории /app лежит файл homework.html , то при запуске
контейнера данный файл должен быть доступен по URL
http://localhost:8000/homework.html );
Работающий с UID 1001

Созданы dockerfile на базе образа alpine, где установлен пакет nginx.

Образ располагается в репозитории https://hub.docker.com/r/oksanazh/hw_web-app

Создан манифест web-pod.yml.

Для применения манифеста используется команда kubectl apply -f web-pod.yml

Запуск приложения проеряется с помощью команды port-forward:

kubectl port-forward --address 0.0.0.0 pod/web 8000:8000

Для проверки работы приложения перейти по ссылке http://localhost:8000/ (http://localhost:8000/index.html)

