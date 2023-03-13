1. Запуск кластера minikube

minikube start

2. Установка в ваш кластер  kubectl debug

brew install aylei/tap/kubectl-debug

3. Запуск agent_daemonset kubectl-debug из манифеста https://raw.githubusercontent.com/aylei/kubectl-debug/dd7e4965e4ae5c4f53e6cf9fd17acc964274ca5c/scripts/agent_daemonset.yml 

kubectl apply -f agent_daemonset.yml 

4. Запускаем pod-ов с приложением web

kubectl apply -f nginx.yml

5. Запуск strace

strace -p 1
strace: Process 1 attached

iptables-tailer

6. Установка netperf оператор

kubectl apply -f crd.yaml
kubectl apply -f rbac.yaml
kubectl apply -f operator.yaml

7. Ззапуск первого теста, применив манифест cr.yaml из папки deploy

kubectl apply -f cr.yaml

kubectl describe netperf.app.example.com/example
Name:         example
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  app.example.com/v1alpha1
Kind:         Netperf
Metadata:
  Creation Timestamp:  2023-03-13T11:19:33Z
  Generation:          4
  Managed Fields:
    API Version:  app.example.com/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2023-03-13T11:19:33Z
    API Version:  app.example.com/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:clientNode:
        f:serverNode:
      f:status:
        .:
        f:clientPod:
        f:serverPod:
        f:speedBitsPerSec:
        f:status:
    Manager:         netperf-operator
    Operation:       Update
    Time:            2023-03-13T11:19:33Z
  Resource Version:  1521
  UID:               94322de1-941d-4abc-9eb4-b8ff81525bce
Spec:
  Client Node:  
  Server Node:  
Status:
  Client Pod:          netperf-client-b8ff81525bce
  Server Pod:          netperf-server-b8ff81525bce
  Speed Bits Per Sec:  10267.3
  Status:              Done
Events:                <none>


iptables-tailer | Тестовое приложение

Добавление сетевой политики для Calico

kubectl apply -f  netperf-calico-policy.yaml

Пересоздание cr

kubectl delete -f netperf-operator/deploy/cr.yaml && kubectl apply -f netperf-operator/deploy/cr.yaml

Тест не прошел:

kubectl describe netperf.app.example.com/example
Name:         example
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  app.example.com/v1alpha1
Kind:         Netperf
Metadata:
  Creation Timestamp:  2023-03-13T11:30:34Z
  Generation:          3
  Managed Fields:
    API Version:  app.example.com/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2023-03-13T11:30:34Z
    API Version:  app.example.com/v1alpha1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:clientNode:
        f:serverNode:
      f:status:
        .:
        f:clientPod:
        f:serverPod:
        f:speedBitsPerSec:
        f:status:
    Manager:         netperf-operator
    Operation:       Update
    Time:            2023-03-13T11:30:34Z
  Resource Version:  2131
  UID:               df76f45e-f781-4e4b-9a8a-6c9a8ec19a80
Spec:
  Client Node:  
  Server Node:  
Status:
  Client Pod:          netperf-client-6c9a8ec19a80
  Server Pod:          netperf-server-6c9a8ec19a80
  Speed Bits Per Sec:  0
  Status:              Started test
Events:                <none>

kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP              NODE       NOMINATED NODE   READINESS GATES
netperf-client-6c9a8ec19a80         1/1     Running   1          3m20s   10.244.120.77   minikube   <none>           <none>
netperf-operator-55b49546b5-5g2vh   1/1     Running   0          15m     10.244.120.69   minikube   <none>           <none>
netperf-server-6c9a8ec19a80         1/1     Running   0          3m22s   10.244.120.76   minikube   <none>   


Подключение к ноде по SSH

счетчики дропов ненулевые

minikube ssh
docker@minikube:~$ sudo iptables --list -nv | grep DROP
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes firewall for dropping marked packets */ mark match 0x8000/0x8000
    0     0 DROP       all  --  *      *      !127.0.0.0/8          127.0.0.0/8          /* block incoming localnet connections */ ! ctstate RELATED,ESTABLISHED,DNAT
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate INVALID
    0     0 DROP       4    --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:_wjq-Yrma8Ly1Svo */ /* Drop IPIP packets from non-Calico hosts */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:fJgusGv3EsAQc-Di */ /* Unknown interface */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:HCO_Y5CCYcy6yW_j */ ctstate INVALID
    0     0 DROP       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:6fwe9eqDVOlMMmLl */ /* Drop VXLAN encapped packets originating in pods */ multiport dports 4789
    0     0 DROP       4    --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:0o27-1Uvvv8xGk2j */ /* Drop IPinIP encapped packets originating in pods */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:VZ6q4dTBNvamoKjS */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:96Ah_DOQcdXKFtrO */ ctstate INVALID
    0     0 DROP       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:yVBOiT7deDA_9s9n */ /* Drop VXLAN encapped packets originating in pods */ multiport dports 4789
    0     0 DROP       4    --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:_bNMdvYQGRh8wCbl */ /* Drop IPinIP encapped packets originating in pods */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:V317GPS6cs5APCSk */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:9Uuw_Yh2EkloZwCW */ ctstate INVALID
    0     0 DROP       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:mORmcsGjlmPHw7zM */ /* Drop VXLAN encapped packets originating in pods */ multiport dports 4789
    0     0 DROP       4    --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:qPr7DIJYITzVn_s8 */ /* Drop IPinIP encapped packets originating in pods */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:Bndq3v2oMkTLcio1 */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:4crT9ecVmmCh1Z4m */ ctstate INVALID
    0     0 DROP       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:ZyQMTEHm_cIdWnns */ /* Drop VXLAN encapped packets originating in pods */ multiport dports 4789
    0     0 DROP       4    --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:NEBaGJGfsMd3Xt2s */ /* Drop IPinIP encapped packets originating in pods */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:TMoMjm8oY4aQGyue */ /* Drop if no policies passed packet */ mark match 0x0/0x20000
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:7npyUCuVwmemkbUO */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:ZMz0eYCLPlz-ea_E */ ctstate INVALID
    0     0 DROP       udp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:b2B6pgtd3uV9tw1p */ /* Drop VXLAN encapped packets originating in pods */ multiport dports 4789
    0     0 DROP       4    --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:HmdoIUpvMoFaz4u3 */ /* Drop IPinIP encapped packets originating in pods */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:tiIlGm5XfnaQ1Lc2 */ /* Drop if no policies passed packet */ mark match 0x0/0x20000
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:RQZJnzm_fHpKwKii */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:SaF3SZhBC-_6Einj */
   21  1260 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:He8TRqGPuUw3VGwk */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:WF2fQAPMaiHHtuut */ /* Unknown interface */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:rUXqkWKb7SdOKjZv */ ctstate INVALID
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:nmSP-ke_5pZChhHc */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:l4WfbTF72a78i4SB */ ctstate INVALID
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:dUI-okRRb0Fgy5Hq */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:K-jC_3IIZRAR2H0v */ ctstate INVALID
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:bOZJGbE7QZ16CYyO */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:QlVXKw9ZNHJlumgF */ ctstate INVALID
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:uDj7k2OlSrpZECly */ /* Drop if no policies passed packet */ mark match 0x0/0x20000
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:frueP68wIL4EQLmC */ /* Drop if no profiles matched */
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:gAZCocaFpK0EQibu */ ctstate INVALID
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:bByn-WXOxbNv7JQb */ /* Drop if no policies passed packet */ mark match 0x0/0x20000
    0     0 DROP       all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:-Z03XziXP92EiL-b */ /* Drop if no profiles matched */


  
    счетчики с действием логирования ненулевые

    sudo iptables --list -nv | grep LOG
    0     0 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:XWC9Bycp2Xf7yVk1 */ LOG flags 0 level 5 prefix "calico-packet: "
   25  1500 LOG        all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* cali:B30DykF1ntLW86eD */ LOG flags 0 level 5 prefix "calico-packet: "

sudo journalctl -k
-- Logs begin at Mon 2023-11-24 11:50:00 UTC, end at Mon 2023-12-24 11:50:07 UTC. --
-- No entries --

Деплой iptaibles-tailer.  

Проверка логов пода iptables-tailer и событий в кластере (kubectl get events -A) ничего не дала

Исправление манифеста используя репозиторий https://github.com/express42/otus-platform-snippets/blob/master/Module-03/Debugging/iptables-tailer.yaml

kubectl apply -f daemonSet.yml


kubectl describe pod netperf-server-913257f58d2c
Name:         netperf-server-913257f58d2c
Namespace:    default
Priority:     0
Node:         minikube/192.168.49.2
Start Time:   Mon, 13 Mar 2023 12:13:28 +0300
Labels:       app=netperf-operator
              netperf-type=server
Annotations:  cni.projectcalico.org/podIP: 10.244.120.93/32
              cni.projectcalico.org/podIPs: 10.244.120.93/32
Status:       Running
IP:           10.244.120.93
IPs:
  IP:           10.244.120.93
Controlled By:  Netperf/example
Containers:
  netperf-server-913257f58d2c:
    Container ID:   docker://a204a69f278528006af3c439c6781bdc8bb095cee85168ece78f357793db43d5
    Image:          tailoredcloud/netperf:v2.7
    Image ID:       docker-pullable://tailoredcloud/netperf@sha256:0361f1254cfea87ff17fc1bd8eda95f939f99429856f766db3340c8cdfed1cf1
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 13 Mar 2023 12:13:28 +0300
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dsvxj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-dsvxj:
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
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  12m   default-scheduler  Successfully assigned default/netperf-server-913257f58d2c to minikube
  Normal  Pulled     12m   kubelet            Container image "tailoredcloud/netperf:v2.7" already present on machine
  Normal  Created    12m   kubelet            Created container netperf-server-913257f58d2c
  Normal  Started    12m   kubelet            Started container netperf-server-913257f58d2
