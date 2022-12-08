Домашнее задание 7


Создан кластер в google-cloud:
default-pool - один узел
infra-pool - три узла

Добавлен taint на default-pool node-role=infra:NoSchedule

Развернут hipster-shop:

kubectl create ns microservices-demo
kubectl apply -f https://raw.githubusercontent.com/express42/otus-platform-snippets/master/Module-02/Logging/microservices-demo-without-resources.yaml -n microservices-demo

Проверка: default-pool k -n microservices-demo get po -o wide
        
Установка ElasticSearch, Fluent Bit, Kibana:

helm repo add elastic https://helm.elastic.co
kubectl create ns observability
helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability
helm upgrade --install kibana elastic/kibana --namespace observability
helm upgrade --install fluent-bit stable/fluent-bit --namespace observability
        
Сервисы поднимаются на ноде из default-pool.

Созданы values для elasticserch kubernetes-logging/elasticsearch.values.yaml с указанием toleration на taint node-role=infra:NoSchedule и nodeSelector для запуска на infra пулах

Обновление с новыми values:

 helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability -f kubernetes-logging/elasticsearch.values.yaml

Установка nginx ingress:

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm upgrade --install nginx-ingress ingress-nginx/ingress-nginx --create-namespace --namespace nginx-ingress --values kubernetes-logging/nginx-ingress.values.yaml
    
доступ к kibana через ingress:
        
helm -n observability upgrade --install kibana elastic/kibana -f kubernetes-logging/kibana.values.yaml

Проверка:
kubectl logs fluent-bit-3gsrt -n logging --tail 3

[ warn] net_tcp_fd_connect: getaddrinfo(host='fluentd'): Name or service not known 
 [error] [out_fw] no upstream connections available '''
        
Установка c указанием имени сервиса elasticsearch-master:

helm upgrade --install fluent-bit stable/fluent-bit --namespace observability --values kubernetes-logging/fluent-bit.values.yaml

Установка prometheus-exporter:
        
helm upgrade --install prometheus-operator prometheus-community/prometheus-operator --namespace=observability --values kubernetes-logging/prometheus.values.yaml
helm upgrade --install elasticsearch-exporter stable/elasticsearch-exporter --set es.uri=http://elasticsearch-master:9200 --set serviceMonitor.enabled=true --namespace=observability

Настроена grafana 

Установлен loki 
