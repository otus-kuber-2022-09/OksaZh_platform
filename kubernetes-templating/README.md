Создан GKE кластер

Подключение:
gcloud beta container clusters get-credentials autopilot-cluster-1 --region us-central1

Установлен helm:

helm version
version.BuildInfo{Version:"v3.9.3", GitCommit:"414ff28d4029ae8c8b05d62aa06c7fe3dee2bc58", GitTreeState:"clean", GoVersion:"go1.17.13"}

1. Установлен nginx-ingress из готовых helm charts:

Добавление репозитория с nginx-ingress chart:

helm repo add stable https://charts.helm.sh/stable 

helm repo list
NAME    URL
stable  https://charts.helm.sh/stable

Создание namespace и release nginx-ingress:

kubectl create ns nginx-ingress

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx --namespace nginx-ingress

Установка cert-mamanger:

kubectl create ns cert-manager

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.crds.yaml -n cert-manager

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml -n cert-manager

Для cert-manager нужно установить Issuer или ClusterIssurer. Используя ClusterIssuer с "Let's Encrypt" в качестве CA. Применена базовая конфигурация:

kubectl apply -f issuer-production.yaml -n cert-manager

Установка chartmuseum

Адрес nginx-ingress:

kubectl get svc -n nginx-ingress

NAME                                 TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.100.0.252   34.116.166.134   80:30994/TCP,443:31721/TCP   64s
ingress-nginx-controller-admission   ClusterIP      10.100.0.69    <none>           443/TCP                      64s

Создан файл values.yaml и переопределена конфигурация nginx-ingress используя TLS и hosts, установлен chartmeuseum с внесёнными изменениями:

kubectl create ns chartmuseum

helm repo add chartmuseum https://chartmuseum.github.io/charts


Проверка chartmuseum в kubernetes:

chartmuseum>helm ls -n chartmuseum

NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
chartmuseum     chartmuseum     1               2022-10-31 16:40:40.6408618 +0300 MSK   deployed        chartmuseum-3.1.0       0.13.1

kubectl get secrets -n chartmuseum
NAME                                TYPE                 DATA   AGE
chartmuseum                         Opaque               0      4m14s
sh.helm.release.v1.chartmuseum.v1   helm.sh/release.v1   1      4m14s


2. Установка harbor:

Создан файл values.yaml и установлен harbor:

helm repo add harbor https://helm.goharbor.io
kubectl create ns harbor
helm upgrade --install harbor harbor/harbor --wait --namespace=harbor --version=2.6.1 -f values.yaml

3. Создание helm chart hipster-shop:

Структура чарта:

helm create kubernetes-templating/hipster-shop

Из папки hipster-shop удален values.yaml и содержимое папки templates. Файл all-hipster-shop.yaml скопирован в папку templates

Запуск и установка hipster-shop:

kubectl create ns hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop

Файл all-hipster-shop.yaml разбит на соответсвтующие компоненты:

Frontend:

helm create kubernetes-templating/frontend

ingress, deployment и service frontend вынесены в kubernetes-templating/frontend. 

Переустановка чарта all-hipster-shop.yaml:

helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop

Установка чарта frontend:

helm upgrade --install frontend kubernetes-templating/frontend --namespace hipster-shop

В чарт frontend добавлен файл values.yaml. Соответствующие значения в deployment.yaml и service.yaml изменены на  на переменные.

В Chart.yaml в all-hipster-shop в качестве зависимости добавлен чарт frontend:

helm delete frontend -n hipster-shop
helm upgrade --install hipster-shop kubernetes-templating/hipster-shop --namespace hipster-shop

4. Kubecfg:

Cервисы paymentservice и shippingservce вынесены отдельно в директорию kubernetes-templating\kubecfg.

Установлен kubecfg.

Создан шаблон service.jsonnet для сервисов, включающий описание service и deployment для shippingservice и deploymentservice.

Проверка, что манифесты генерируются корректно:

kubecfg show services.jsonnet

Установка:

kubecfg update services.jsonnet --namespace hipster-shop

5. Kustomize:

Установлен kustomize:

Удалено описание сервиса recommendationservice из all-hipster-shop.yaml и вынесено в директорию kubernetes-templating/kustomize/base.

Проверка, что YAML с манифестамы генерируется валидным:

kustomize build kubernetes-templating/kustomize/base/

Созданы дирректории kubernetes-templating/kustomize/overriddes/hipster-shop(-prod) куда добавлены файлы с кастомизацией, которые переписывают переменные (такие как labels) для манифестов в зависимости от среды:

kubectl create namespace hipster-shop-prod

kubectl apply -k kubernetes-templating/kustomize/overrides/hispter-shop/
kubectl apply -k kubernetes-templating/kustomize/overrides/hispter-shop-prod/