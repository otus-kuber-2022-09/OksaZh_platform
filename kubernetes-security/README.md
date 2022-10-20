Задание 1.

1.1 Создать Service Account bob , дать ему роль admin в рамках всего
кластера

Создан манифест 01-bob.yaml для содания Service Account и 02-cluster-role-binding.yaml для выдачи прав аккаунту bob в рамках кластера.

Применение манифестов:

kubectl apply -f 01-bob.yaml
kubectl apply -f 02-cluster-role-binding.yaml

Проверка прав:

rakkess-amd64-windows>rakkess-amd64-windows --sa default:bob

NAME                                                          LIST  CREATE  UPDATE  DELETE
apiservices.apiregistration.k8s.io                            ✖     ✖       ✖       ✖
bindings                                                            ✖
certificatesigningrequests.certificates.k8s.io                ✖     ✖       ✖       ✖
clusterrolebindings.rbac.authorization.k8s.io                 ✖     ✖       ✖       ✖
clusterroles.rbac.authorization.k8s.io                        ✖     ✖       ✖       ✖
componentstatuses                                             ✖
configmaps                                                    ✔     ✔       ✔       ✔
controllerrevisions.apps                                      ✔     ✖       ✖       ✖
cronjobs.batch                                                ✔     ✔       ✔       ✔
csidrivers.storage.k8s.io                                     ✖     ✖       ✖       ✖
csinodes.storage.k8s.io                                       ✖     ✖       ✖       ✖
csistoragecapacities.storage.k8s.io                           ✖     ✖       ✖       ✖
customresourcedefinitions.apiextensions.k8s.io                ✖     ✖       ✖       ✖
daemonsets.apps                                               ✔     ✔       ✔       ✔
deployments.apps                                              ✔     ✔       ✔       ✔
endpoints                                                     ✔     ✖       ✖       ✖
endpointslices.discovery.k8s.io                               ✔     ✖       ✖       ✖
events                                                        ✔     ✔       ✔       ✔
events.events.k8s.io                                          ✖     ✖       ✖       ✖
flowschemas.flowcontrol.apiserver.k8s.io                      ✖     ✖       ✖       ✖
horizontalpodautoscalers.autoscaling                          ✔     ✔       ✔       ✔
ingressclasses.networking.k8s.io                              ✖     ✖       ✖       ✖
ingresses.networking.k8s.io                                   ✔     ✔       ✔       ✔
jobs.batch                                                    ✔     ✔       ✔       ✔
leases.coordination.k8s.io                                    ✖     ✖       ✖       ✖
limitranges                                                   ✔     ✖       ✖       ✖
localsubjectaccessreviews.authorization.k8s.io                      ✔
mutatingwebhookconfigurations.admissionregistration.k8s.io    ✖     ✖       ✖       ✖
namespaces                                                    ✔     ✖       ✖       ✖
networkpolicies.networking.k8s.io                             ✔     ✔       ✔       ✔
nodes                                                         ✖     ✖       ✖       ✖
persistentvolumeclaims                                        ✔     ✔       ✔       ✔
persistentvolumes                                             ✖     ✖       ✖       ✖
poddisruptionbudgets.policy                                   ✔     ✔       ✔       ✔
pods                                                          ✔     ✔       ✔       ✔
podsecuritypolicies.policy                                    ✖     ✖       ✖       ✖
podtemplates                                                  ✖     ✖       ✖       ✖
priorityclasses.scheduling.k8s.io                             ✖     ✖       ✖       ✖
prioritylevelconfigurations.flowcontrol.apiserver.k8s.io      ✖     ✖       ✖       ✖
replicasets.apps                                              ✔     ✔       ✔       ✔
replicationcontrollers                                        ✔     ✔       ✔       ✔
resourcequotas                                                ✔     ✖       ✖       ✖
rolebindings.rbac.authorization.k8s.io                        ✔     ✔       ✔       ✔
roles.rbac.authorization.k8s.io                               ✔     ✔       ✔       ✔
runtimeclasses.node.k8s.io                                    ✖     ✖       ✖       ✖
secrets                                                       ✔     ✔       ✔       ✔
selfsubjectaccessreviews.authorization.k8s.io                       ✔
selfsubjectrulesreviews.authorization.k8s.io                        ✔
serviceaccounts                                               ✔     ✔       ✔       ✔
services                                                      ✔     ✔       ✔       ✔
statefulsets.apps                                             ✔     ✔       ✔       ✔
storageclasses.storage.k8s.io                                 ✖     ✖       ✖       ✖
subjectaccessreviews.authorization.k8s.io                           ✖
tokenreviews.authentication.k8s.io                                  ✖
validatingwebhookconfigurations.admissionregistration.k8s.io  ✖     ✖       ✖       ✖
volumeattachments.storage.k8s.io                              ✖     ✖       ✖       ✖

1.2 Создать Service Account dave без доступа к кластеру

Создан манифест 03-dave.yaml для содания Service Account.

Применение манифеста:

kubectl apply -f 03-dave.yaml

Проверка прав:

rakkess-amd64-windows --sa default:dave

NAME                                                          LIST  CREATE  UPDATE  DELETE
apiservices.apiregistration.k8s.io                            ✖     ✖       ✖       ✖
bindings                                                            ✖
certificatesigningrequests.certificates.k8s.io                ✖     ✖       ✖       ✖
clusterrolebindings.rbac.authorization.k8s.io                 ✖     ✖       ✖       ✖
clusterroles.rbac.authorization.k8s.io                        ✖     ✖       ✖       ✖
componentstatuses                                             ✖
configmaps                                                    ✖     ✖       ✖       ✖
controllerrevisions.apps                                      ✖     ✖       ✖       ✖
cronjobs.batch                                                ✖     ✖       ✖       ✖
csidrivers.storage.k8s.io                                     ✖     ✖       ✖       ✖
csinodes.storage.k8s.io                                       ✖     ✖       ✖       ✖
csistoragecapacities.storage.k8s.io                           ✖     ✖       ✖       ✖
customresourcedefinitions.apiextensions.k8s.io                ✖     ✖       ✖       ✖
daemonsets.apps                                               ✖     ✖       ✖       ✖
deployments.apps                                              ✖     ✖       ✖       ✖
endpoints                                                     ✖     ✖       ✖       ✖
endpointslices.discovery.k8s.io                               ✖     ✖       ✖       ✖
events                                                        ✖     ✖       ✖       ✖
events.events.k8s.io                                          ✖     ✖       ✖       ✖
flowschemas.flowcontrol.apiserver.k8s.io                      ✖     ✖       ✖       ✖
horizontalpodautoscalers.autoscaling                          ✖     ✖       ✖       ✖
ingressclasses.networking.k8s.io                              ✖     ✖       ✖       ✖
ingresses.networking.k8s.io                                   ✖     ✖       ✖       ✖
jobs.batch                                                    ✖     ✖       ✖       ✖
leases.coordination.k8s.io                                    ✖     ✖       ✖       ✖
limitranges                                                   ✖     ✖       ✖       ✖
localsubjectaccessreviews.authorization.k8s.io                      ✖
mutatingwebhookconfigurations.admissionregistration.k8s.io    ✖     ✖       ✖       ✖
namespaces                                                    ✖     ✖       ✖       ✖
networkpolicies.networking.k8s.io                             ✖     ✖       ✖       ✖
nodes                                                         ✖     ✖       ✖       ✖
persistentvolumeclaims                                        ✖     ✖       ✖       ✖
persistentvolumes                                             ✖     ✖       ✖       ✖
poddisruptionbudgets.policy                                   ✖     ✖       ✖       ✖
pods                                                          ✖     ✖       ✖       ✖
podsecuritypolicies.policy                                    ✖     ✖       ✖       ✖
podtemplates                                                  ✖     ✖       ✖       ✖
priorityclasses.scheduling.k8s.io                             ✖     ✖       ✖       ✖
prioritylevelconfigurations.flowcontrol.apiserver.k8s.io      ✖     ✖       ✖       ✖
replicasets.apps                                              ✖     ✖       ✖       ✖
replicationcontrollers                                        ✖     ✖       ✖       ✖
resourcequotas                                                ✖     ✖       ✖       ✖
rolebindings.rbac.authorization.k8s.io                        ✖     ✖       ✖       ✖
roles.rbac.authorization.k8s.io                               ✖     ✖       ✖       ✖
runtimeclasses.node.k8s.io                                    ✖     ✖       ✖       ✖
secrets                                                       ✖     ✖       ✖       ✖
selfsubjectaccessreviews.authorization.k8s.io                       ✔
selfsubjectrulesreviews.authorization.k8s.io                        ✔
serviceaccounts                                               ✖     ✖       ✖       ✖
services                                                      ✖     ✖       ✖       ✖
statefulsets.apps                                             ✖     ✖       ✖       ✖
storageclasses.storage.k8s.io                                 ✖     ✖       ✖       ✖
subjectaccessreviews.authorization.k8s.io                           ✖
tokenreviews.authentication.k8s.io                                  ✖
validatingwebhookconfigurations.admissionregistration.k8s.io  ✖     ✖       ✖       ✖
volumeattachments.storage.k8s.io                              ✖     ✖       ✖       ✖
No namespace given, this implies cluster scope (try -n if this is not intended)

Задание 2.

2.1 Создать Namespace prometheus

Создан манифест 01-ns-prometheus.yaml

2.2. Создать Service Account carol в этом Namespace

Создан манифест 02-sa-carol.yaml

2.3 Дать всем Service Account в Namespace prometheus возможность делать
get , list , watch в отношении Pods всего кластера

Создан 03-cluster-role-view-pod.yaml для выполнения get , list , watch в рамках Pods кластера и манифест 04-cluster-role-binding.yaml для назначения прав Service Account carol

Проверка:

kubectl auth can-i list pods --as system:serviceaccount:prometheus:carol
yes

Задание 3.

3.1. Создать Namespace dev

Создан манифест 01-ns-dev.yaml

3.2 Создать Service Account jane в Namespace dev

Создан манифест 02-sa-jane-ken.yaml для создания Service Account jane 

3.3 Дать jane роль admin в рамках Namespace dev

Создан манифест 03-roles-in-namespace.yaml для назначения роли admin Service Account jane

3.4. Создать Service Account ken в Namespace dev

Создан манифест 02-sa-jane-ken.yaml для создания Service Account ken 

3.5 Дать ken роль view в рамках Namespace dev

Создан манифест 03-roles-in-namespace.yaml для назначения роли view Service Account ken

Проверка после применения манифестов:

kubectl auth can-i list pods --as system:serviceaccount:prometheus:carol
yes

kubectl auth can-i create deployments --as system:serviceaccount:dev:jane -n dev
yes

kubectl auth can-i create deployments --as system:serviceaccount:dev:ken -n dev
no

kubectl auth can-i get deployments --as system:serviceaccount:dev:ken -n dev
yes