Домашнее задание 12

1. kubeadm 

Созданы ноды для кластера:

master - 1 экземпляр (n1-standard-2)
worker - 3 экземпляра (n1-standard-1)

Подготовлены машины 
Отключен swap:

swapoff -a
vi /etc/fstab

Включаем маршрутизацию

cat > /etc/sysctl.d/99-kubernetes-cri.conf <<eof net.bridge.bridge-nf-call-
iptables="1" net.ipv4.ip_forward="1" net.bridge.bridge-nf-call-ip6tables="1" eof=""
sysctl="" --system="" <="" code=""></eof>

Установим docker:

apt-get update && apt-get install -y \
apt-transport-https ca-certificates curl software-properties-common gnupg2

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

apt-get update && apt-get install -y \
containerd.io=1.2.13-1 \
docker-ce=5:19.03.8~3-0~ubuntu-$(lsb_release -cs) \
docker-ce-cli=5:19.03.8~3-0~ubuntu-$(lsb_release -cs)

cat > /etc/docker/daemon.json <<eof {="" "exec-opts":=""
["native.cgroupdriver="systemd"]," "log-driver":="" "json-file",="" "log-opts":=""
"max-size":="" "100m"="" },="" "storage-driver":="" "overlay2"="" }="" eof=""
mkdir="" -p="" etc="" systemd="" system="" docker.service.d="" #="" restart=""
docker.="" systemctl="" daemon-reload="" docker<="" code=""></eof>


Установка kubeadm, kubelet and kubectl
Установим версию 1.17.4, данные команды необходимо выполнить на всех
нодах 

apt-get update && apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<eof> /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet=1.17.4-00 kubeadm=1.17.4-00 kubectl=1.17.4-00</eof>


Создание кластера

Создадим настроим мастер ноду при помощи kubeadm, для этого на ней
выполним:

kubeadm init --pod-network-cidr=192.168.0.0/24

Копируем конфиг kubectl

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Проверяем:

kubectl get nodes


Устанавливаем сетевой плагин

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Подключаем worker-ноды

kubeadm token list

Получить хеш

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform
der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'

kubectl get nodes
NAME      STATUS   ROLES    AGE     VERSION
master    Ready    master   9m51s   v1.17.4
worker1   Ready    <none>   3m6s    v1.17.4
worker2   Ready    <none>   89s     v1.17.4
worker3   Ready    <none>   64s     v1.17.4

Запуск нагрузки
Для демонстрации работы кластера запустим nginx


apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deployment
spec:
selector:
matchLabels:
app: nginx
replicas: 4
template:
metadata:
labels:
app: nginx
spec:
containers:
- name: nginx
image: nginx:1.17.2
ports:
- containerPort: 80

kubectl apply -f deployment.yaml

Обновление кластера

Обновление мастера

Допускается, отставание версий worker-нод от master, но не наоборот. Поэтому обновление будем начинать с нее master-нода у нас версии 1.16.8. Обновление пакетов

apt-get update && apt-get install -y kubeadm=1.18.0-00 \
kubelet=1.18.0-00 kubectl=1.18.0-00

Проверка

kubectl get nodes
NAME      STATUS   ROLES    AGE     VERSION
master    Ready    master   16m     v1.18.0
worker1   Ready    <none>   9m55s   v1.17.4
worker2   Ready    <none>   7m25s   v1.17.4
worker3   Ready    <none>   6m18s   v1.17.4


Обновим остальные компоненты кластера
Обновление компонентов кластера (API-server, kube-proxy, controller-
manager)

Проверка

# просмотр изменений, которые собирает сделать kubeadm
kubeadm upgrade plan
# применение изменений
kubeadm upgrade apply v1.18.0

kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2022-12-28T11:56:30Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
kubelet --version
Kubernetes v1.18.0

kubectl version

Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2022-12-28T11:30:40Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2022-12-28T11:33:45Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}


kubectl describe pod kube-apiserver-master-instance-0 -n kube-system

Вывод worker-нод из планирования

Первым делом, мы сливаем всю нагрузку с ноды и выводим ее из
планирования:

kubectl drain worker-instance-1

kubectl get nodes -o wide
NAME                STATUS                     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
worker-instance-0   Ready,SchedulingDisabled   <none>   22m   v1.17.4   10.128.0.48   <none>        Ubuntu 

к статусу добавилась строчка SchedulingDisabled


Обновление worker-нод
На worker-ноде выполняем
apt-get install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
systemctl restart kubelet

После обновления kubectl показывает новую версию, и статус SchedulingDisabled

kubectl get nodes
NAME      STATUS                     ROLES    AGE   VERSION
master    Ready                      master   24m   v1.18.0
worker1   Ready,SchedulingDisabled   <none>   19m   v1.18.0
worker2   Ready                      <none>   16m   v1.17.4
worker3   Ready                      <none>   15m   v1.17.4

Возвращение ноды в планирование

kubectl uncordon worker-instance-1

Обновляем оставшиеся ноды:

kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
master    Ready    master   48m   v1.18.0
worker1   Ready    <none>   42m   v1.18.0
worker2   Ready    <none>   39m   v1.18.0
worker3   Ready    <none>   38m   v1.18.0


2. Автоматическое развертывание кластеров

# получение kubespray
git clone https://github.com/kubernetes-sigs/kubespray.git
# установка зависимостей
sudo pip install -r requirements.txt
# копирование примера конфига в отдельную директорию
cp -rfp inventory/sample inventory/mycluster

inventory.ini - добавляем IP адреса своих нод 

После редактирования конфига можно устанавливать кластер:
ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root \
--user=${SSH_USERNAME} --key-file=${SSH_PRIVATE_KEY} cluster.yml



