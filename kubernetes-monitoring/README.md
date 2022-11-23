Домашнее задание 7

Создан кастомный образ nginx:

Создан Dockerfile на базе образа из kubernetes-intro. Добавлена домашняя страница и конфигурация для сора базовых метрик.

Запуск и проверка:

docker run -d -p 8000:8000 oksanazh/nginx:v0.3

$  curl http://localhost:8000
<html>
    <head>Homework 7</head>
    <body>
        <p>Nginx for monitoring</p>
    </body>
</html>

curl http://localhost:8000/basic_status
Active connections: 1 
server accepts handled requests
 2 2 2 
Reading: 0 Writing: 1 Waiting: 0 

Создание кластера kind

cd kubernetes-monitoring
kind create cluster --config cluster.yml

Установка мониторинга
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create ns prometheus
helm install prometheus-operator prometheus-community/kube-prometheus-stack --namespace=prometheus --wait -f values.yml

Запустим deployment с nginx, service и servicemonitor

kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f servicemonitor.yml

Порты в графану:

kubectl port-forward  --namespace prometheus svc/prometheus-operator-grafana 8000:80

Импорт grafana-dashboard.json,что доступна по http://localhost:8000

Порты в service nginx:

kubectl port-forward svc/nginx 8080:80

curl http://localhost:8080