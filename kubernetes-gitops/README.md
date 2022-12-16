Домашнее задание 10

В качестве Репозитория и CI-системы в используется GitLab.

Создан в GitLab публичный проект microservices-demo.

Подготовка GitLab репозитория:

В проект microservices-demo перемещен код из GitHub репозитория с демо сайтом. 

git clone https://github.com/GoogleCloudPlatform/microservices-demo
cd microservices-demo

git config user.name "Oksana Zherebtsova"
git config user.email "ksundelsun@live.com"

git remote add gitlab git@gitlab.com:oksaZ/microservices-demo.git
git remote remove origin
git push gitlab master

Создание Helm чартов

Используются Helm чарты из готового демонстранционного репозиотрия git@gitlab.com:express42/kubernetes-platform-demo/microservices-demo.git

Параметризованы название образа и его тег:

image:
  repository: adservice
  tag: latest

Результаты размещены в директории deploy/charts


Создан кластер с 4-мя нодами типа n1-standard-2

Continuous Integration

Собраны Docker образы для всех микросервисов и размещены в Docker Hub:

cd ./src
for d in *; do
    cd "$(pwd)/$d";
    docker build  -t oksanazh/$d:v0.0.1 .
    docker push oksanazh/$d:v0.0.1
    cd ..;
done

Создан pipeline, который выполняет стадии:

- Сборку Docker образа для каждого из микросервисов
- Push данного образа в Docker Hub

https://gitlab.com/OksaZ/microservices-demo/-/blob/main/.gitlab-ci.yml


GitOps. Подготовка.

Установка CRD, добавляющую в кластер новый ресурс - HelmRelease:

kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml

Установка Flux

helm repo add fluxcd https://charts.fluxcd.io
kubectl create namespace flux
helm upgrade --install flux fluxcd/flux -f flux.values.yaml --namespace flux

Создан flux.values.yaml на основе указанного файла изменив URL со значениями репзитория. 

Установка Flux в кластер, в namespace flux:

kubectl create namespace flux
helm upgrade --install flux fluxcd/flux -f flux.values.yaml --namespace flux

Установка Helm operator (значения взяты из указанного файла):

helm upgrade --install helm-operator fluxcd/helm-operator -f helm-operator.values.yaml --namespace flux

Для frontend установлен Service monitor

Проверка корректности работы Flux. Flux должен автоматически синхронизировать состояние кластера и репозитория. Это касается не только сущностей HelmRelease, но и обыкновенных манифестов:

Создан манифест microservices-demo и сделан push в репозиторий

Через некоторое время появился новый namespaces

kubectl get namespaces
NAME                 STATUS   AGE
default              Active   3h3m
flux                 Active   45m
kube-node-lease      Active   3h4m
kube-public          Active   3h4m
kube-system          Active   3h4m
microservices-demo   Active   54s

HelmRelease

Для описания сущностей такого вида создана отдельнуя директория deploy/releases и добавлен файл frontend.yaml с описанием конфигурации релиза.

Выполнен push в репозиторий

fronend pod появился в namespace microservices-demo:

kubectl get helmrelease -n microservices-demo
NAME       RELEASE    STATUS     MESSAGE                       AGE
frontend   frontend   deployed   Helm release sync succeeded   10m

Обновление образа

Внесены изменения в номер версии frontend до тега v0.0.2

image.png


ts=2022-11-22T17:53:39.449727104Z caller=helm.go:69 component=helm version=v3 info="Created a new Deployment called \"frontend-hipster\" in microservices-demo\n" targetNamespace=microservices-demo release=frontend
ts=2022-11-22T17:53:39.622575038Z caller=helm.go:69 component=helm version=v3 info="Deleting \"frontend\" in microservices-demo..." targetNamespace=microservices-demo release=frontend
ts=2022-11-22T17:53:39.706761483Z caller=helm.go:69 component=helm version=v3 info="updating status for upgraded release for frontend" targetNamespace=microservices-demo release=frontend
ts=2022-11-22T17:53:39.791761524Z caller=release.go:364 component=release release=frontend targetNamespace=microservices-demo resource=microservices-demo:helmrelease/frontend helmVersion=v3 info="upgrade succeeded" revision=a9916208cad7175a2fb6afe9d9b787759c22f511 phase=upgrade


Обновление Helm chart

Внесены изменения в Helm chart frontend , изменено имя  deployment на frontend-hipster

В логах helm-operator появились строки, указывающие на механизм проверки изменений в Helm chart и определения необходимости обновить релиз:

ts=2022-12-16T11:10:17.289472552Z caller=release.go:360 component=release release=frontend targetNamespace=microservices-demo resource=microservices-demo:helmrelease/frontend helmVersion=v3 info="performing dry-run upgrade to see if release has diverged"
ts=2022-12-16T11:10:17.291632571Z caller=helm.go:69 component=helm version=v3 info="preparing upgrade for frontend" targetNamespace=microservices-demo release=frontend
ts=2022-12-16T11:10:17.944814591Z caller=helm.go:69 component=helm version=v3 info="performing update for frontend" targetNamespace=microservices-demo release=frontend
ts=2022-12-16T11:10:17.962387822Z caller=helm.go:69 component=helm version=v3 info="dry run for frontend" targetNamespace=microservices-demo release=frontend
ts=2022-12-16T11:10:17.967692289Z caller=release.go:404 component=release release=frontend targetNamespace=microservices-demo resource=microservices-demo:helmrelease/frontend helmVersion=v3 info="no changes" action=skip


Самостоятельное задание

Добавлены манифесты HelmRelease для всех микросервисов входящих в состав HipsterShop
Проверено, что все микросервисы успешно развернулись в Kubernetes кластере

kubectl get helmrelease -n microservices-demo

NAME                      RELEASE                   STATUS     MESSAGE                       AGE
adservice                 adservice                 deployed   Helm release sync succeeded   59s
cartservice               cartservice               deployed   Helm release sync succeeded   59s
checkoutservice           checkoutservice           deployed   Helm release sync succeeded   59s
currencyservice           currencyservice           deployed   Helm release sync succeeded   59s
emailservice              emailservice              deployed   Helm release sync succeeded   59s
frontend                  frontend                  deployed   Helm release sync succeeded   9m10s
grafana-load-dashboards   grafana-load-dashboards   deployed   Helm release sync succeeded   59s
loadgenerator             loadgenerator             deployed   Helm release sync succeeded   59s
paymentservice            paymentservice            deployed   Helm release sync succeeded   59s
productcatalogservice     productcatalogservice     deployed   Helm release sync succeeded   59s
recommendationservice     recommendationservice     deployed   Helm release sync succeeded   59s
shippingservice           shippingservice     deployed   Helm release sync succeeded   59s


Canary deployments с  Flagger и IstioFlagger и Istio

Установлен Istio согласно документации
Установлен Flagger

Добавление helm-репозитория flagger:

helm repo add flagger https://flagger.app

Установка CRD для Flagger:

kubectl apply -f https://raw.githubusercontent.com/weaveworks/flagger/master/artifacts/flagger/crd.yaml

Установка flagger с указанием использовать Istio:

helm upgrade --install flagger flagger/flagger \
--namespace=istio-system \
--set crd.create=false \
--set meshProvider=istio \
--set metricsServer=http://prometheus:9090

Istio | Sidecar Injection

Изменено созданное ранее описание namespace microservices-demo

kubectl get ns microservices-demo --show-labels
NAME                 STATUS   AGE   LABELS
microservices-demo   Active   26h   fluxcd.io/sync-gc-mark=sha256.Y_9QDTGdvbnt-RT4AOYggq9aT5JxXxTUYNFsQgCg3un8,istio-injection=enabled,kubernetes.io/metadata.name=microservices-demo

Istio | Sidecar Injection

kubectl delete pods --all -n microservices-demo

Проверка, что контейнер с названием istio-proxy появился внутри каждого pod

kubectl delete pods --all -n microservices-demo

kubectl describe pod -l app=frontend -n microservices-demo
Containers:
  server:
    Container ID:   containerd://59a91ae04eb3bc4ccee88364670f7b99c28ef53871cedecaf9afaed872c9cd8e
    Image:          oksanazh/frontend:v0.0.2
    Image ID:       docker.io/oksanazh/frontend@sha256:a24767d32f0f0cab3315efdbda24489dd66a001e69cd83e3dfbcde8f9ffd4b34
  istio-proxy:
    Container ID:  containerd://dd1527fffd18ad725948bf3f502f4f2ea05970c3f8dbeab165b0a76a36296a7a
    Image:         docker.io/istio/proxyv2:1.16.0
    Image ID:      docker.io/istio/proxyv2@sha256:f6f97fa4fb77a3cbe1e3eca0fa46bd462ad6b284c129cf57bf91575c4fb50cf9
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11m   default-scheduler  Successfully assigned microservices-demo/frontend-4ff66tg648-321rv to gke-ivory-labs-default-pool-t537vg6-7mnl
  Normal  Pulled     11m   kubelet            Container image "docker.io/istio/proxyv2:1.16.0" already present on machine
  Normal  Created    11m   kubelet            Created container istio-init
  Normal  Started    11m   kubelet            Started container istio-init
  Normal  Pulling    11m   kubelet            Pulling image "oksanazh/frontend:v0.0.2"
  Normal  Pulled     11m   kubelet            Successfully pulled image "oksanazh/frontend:v0.0.2" in 9.131442395s
  Normal  Created    11m   kubelet            Created container server
  Normal  Started    11m   kubelet            Started container server
  Normal  Pulled     11m   kubelet            Container image "docker.io/istio/proxyv2:1.16.0" already present on machine
  Normal  Created    11m   kubelet            Created container istio-proxy
  Normal  Started    11m   kubelet            Started container istio-proxy

  Доступ к frontend

kubectl get gateway -n microservices-demo
NAME               AGE
frontend           15m4s

EXTERNAL-IP сервиса istio-ingressgateway:

kubectl get svc istio-ingressgateway -n istio-system

Istio | Самостоятельное задание

Манифесты Gateway и VirtualService перемещены в Helm chart frontend.

    gateway.yaml
    virtualService.yaml

Оригинальные манифесты удалены вместе с директорией deploy/istio

Flagger | Canary

Добавлен в Helm chart frontend еще один файл - canary.yaml В нем будем хранить описание стратегии, по которой необходимо обновлять данный микросервис.

После установки Flagger появились PODы в namespace istio-system

kubectl get canary -n microservices-demo

NAME       STATUS        WEIGHT   LASTTRANSITIONTIME
frontend   Initialized   0        2022-12-16T12:27:24Z

kubectl get pods -n microservices-demo -l app=frontend-primary
NAME                                READY   STATUS    RESTARTS   AGE
frontend-primary-684f56c555-wvbsd   2/2     Running   0          13m

Соберан новый образ frontend с тегом v0.0.3 и выполнен push в Docker Hub

Через некоторое время в выводе kubectl describe canary frontend -n microservices-demo появилось следующее:

describe canary frontend -n microservices-demo

Events:
Type     Reason  Age                From     Message
----     ------  ----               ----     -------
Warning  Synced  43m                flagger  IsPrimaryReady failed: primary daemonset frontend-primary.microservices-demo not ready: waiting for rollout to finish: observed deployment generation less then desired generation
Normal   Synced  42m                flagger  Initialization done! frontend.microservices-demo
Normal   Synced  13m                flagger  New revision detected! Scaling up frontend.microservices-demo
Normal   Synced  12m                flagger  Starting canary analysis for frontend.microservices-demo
Normal   Synced  12m                flagger  Advance frontend.microservices-demo canary weight 5
Warning  Synced  10m (x5 over 12m)  flagger  Halt advancement no values found for istio metric request-duration probably frontend.microservices-demo is not receiving traffic
Warning  Synced  9m50s              flagger  Rolling back frontend.microservices-demo failed checks threshold reached 5
Warning  Synced  9m50s              flagger  Canary failed! Scaling down frontend.microservices-demo


Это произошло из-за не верное настроеного load-generator (который настроен на внутренний адрес) и не хватало метрик для анализа релиза.

Изменен values.yaml указав правильный ingress хост для loadgenerator, push. Проверка:

kubectl describe canary frontend -n microservices-demo

Events:
Type Reason Age From Message
---- ------ ---- ---- -------
Normal Synced 8m27s flagger New revision detected! Scaling up
frontend.microservices-demo
Normal Synced 7m58s flagger Starting canary analysis for frontend.microservicesdemo
Normal Synced 7m57s flagger Advance frontend.microservices-demo canary weight 5
Normal Synced 6m27s flagger Advance frontend.microservices-demo canary weight 10
Normal Synced 5m57s flagger Advance frontend.microservices-demo canary weight 15
Normal Synced 5m27s flagger Advance frontend.microservices-demo canary weight 20
Normal Synced 3m57s flagger Advance frontend.microservices-demo canary weight 25
Normal Synced 3m27s flagger Advance frontend.microservices-demo canary weight 30