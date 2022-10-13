Домашнее задание 4

1. В описание манифеста web-pod.yml добавлен readinessProbe:

          readinessProbe:
            httpGet:
              path: /index.html
              port: 8000

Запущен pod:

kubectl apply -f web-pod.yaml

Проверка статуса pod:

kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
web    0/1     Running   0          9s

Pod не перешёл в состояние Ready т.к. не прошел проверку готовности(указан port - 80 вместо 8000).

Добавлен livenessProbe:

livenessProbe:
    tcpSocket:
      {port: 8080}

2. На основе kuberneted-intro\web-pod.yaml создан Deployment с измененными readinessProbe и кол-вом реплик

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      name: web
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: oksanazh/hw_web-app:1.0.0
          readinessProbe:
            httpGet:
              path: /index.html
              port: 8000
          livenessProbe:
            tcpSocket: { port: 8000 }

Применение манифеста:

kubectl apply -f web-deploy.yaml

Описание deployment:

kubectl describe deploy/web

В Condifitions Deployment - Available and Progressing"

Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable

В манифест добалвена стратегия развертывания:

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 100%

При использовании различных вариаций значений параметров maxUnavailable и maxSurge можно получить различные стратегии обновления подов:
Canary  - maxUnavailable: 1 \maxSurge: 0
Blue-Green - maxUnavailable: 0 \maxSurge: 100%
удалить все pod и затем создать новые - maxUnavailable: 100% \maxSurge: 0

3. Создадн манифест для Service с типом ClusterIP: 

kubectl apply -f web-svc-cip.yaml
kubectl get service
NAME          TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
kubernetes    ClusterIP      10.96.0.1        <none>         443/TCP        153m
web-svc-cip   ClusterIP      10.103.215.124   <none>         80/TCP         94m

Проверен доступ к  ClusterIP

minikube ssh
curl http://0.103.215.124 /index.html
ping 0.103.215.124 
arp -an
ip addr show
sudo iptables --list -nv -t nat

4. Установлен IPVS  и настроен minikube для его использования
5. Установлен MetalLB. Провекра установки:

kubectl --namespace metallb-system get all
6. Настроен MetalLB с использованием ConfigMap (манифест metallb-config.yaml)
7. Создан  Service с MetalLB в качетве LoadBalancer (манифест web-svc-lb.yaml)

kubectl --namespace metallb-system logs pod/controller-7696f658c8-bxp7q:

{"caller":"service.go:114","event":"ipAllocated","ip":"172.17.255.1","msg":"IP address assigned by controller","service":"default/web-svc-lb","ts":"2022-10-13T08:25:14.784174051Z"}

IP 172.17.255.1 назначен сервису web-svc-lb

kubectl describe svc web-svc-lb:

IP:                       10.96.125.130
IPs:                      10.96.125.130
LoadBalancer Ingress:     172.17.255.1

8. создан маршрут с minikube на хост:

minikube ip

172.17.6.227

ip route add 172.17.255.0/24 via 172.17.6.227

Проверен доступ http://172.17.255.1/index.html

9. Создан ingress

Основной манифест:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

10/ Создан манифест nginx-lb.yaml с конфигурацией балансировщика нагрузки. 

Получен ip адрес

kubectl get svc -n ingress-nginx

NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)                      AGE
ingress-nginx                        LoadBalancer   10.110.207.133   172.17.255.3   80:30964/TCP,443:30271/TCP   86m

10. Создан headless-сервис для веб-приложения (web-svc-headless.yaml)

kubectl apply -f web-svc-headless.yaml

11. Создан манифест web-ingress.yaml 

kubectl describe ingress/web:

Name:             web
Namespace:        default
Address:          172.17.255.3
Default backend:  default-http-backend:80 (<none>)
Rules:
Host  Path  Backends
----  ----  --------
*
        /web   web-svc:8000 (172.17.0.7:8000,172.17.0.8:8000,172.17.0.9:8000)