# stepik-k8s
Конспект по курсу "Kubernetes для начинающих + практический опыт" на Stepik

### Архитектура
Нода - физическая или виртуальная машина, на которой установлен k8s
Master нода - управляющая нода кластера
Worker нода - компьютер на котором k8s запускает наши контейнеры

### Состав k8s:
API-сервер
Container Runtime - в нашем случае докер, но может быть и другая среда управления контейнерами
Controller - отслеживание состояния нод
Scheduler - распределяет контейнеры на ноды
Kubelet - агент, который работает на каждом узле кластера, отвечающий за то, чтобы контейнеры работали должным образом
ETCD - БД (ключ-значение) содержащая всю инфу о кластере

### Первые команды
```bash
# Развертывание приложения внутри кластера
kubectrl run hello-minikube
# Просмотр информации о кластере
kubectl cluster-info 
# Вывод списка все узлов в кластере
kubectl get nodes
```

### Docker и ContainerD
```bash
# containerD - runtime для контейнеров. Позволяет запускать контейнеры без Docker

# ctr
# Работает с containerD
ctr images pull docker.io/library/nginx:alpine
ctr run docker.io/library/nginx

# nerdctl
# nerdctl - примерно тоже самое, что и docker cli + доп. фишки
# Работает с containerD
nerdctl run --name nginx nginx:alpine
nerdctl run -d --name redis -p 6379:6379

# crictl - не привязана к конкретному движку. Работает со всеми современными средствами исполнения. Ставится отдельно. Не идет в комплекте с runtime
# Работает со всеми CRI (Conjtainer Runtime Interface) средами исполнения
# Принадлежит разработчикам k8s
crictl pull alpine
crictl images
crictl ps -a
crictl exec -i -t 3db02e25bce72 hostname
crictl logs 3db02e25bce72
crictl pods # может работать с подами k8s
```

### Базовые команды
```
kubectl get nodes
kubectl get nodes -owide
```

### Minikube
```bash
# Существует множетство способов установки и настройки k8s
# Minikube, MicroK8s, Kubeadm
# AWS, GCP, Azure
# play-with-k8s.com, katacoda.com, rotoro.cloud

# Minikube - предварительно настроенный одноузловой кластер kubernetes

# Установка kubectl
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt-get install -y kubectl

# Установка minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube

sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/

# Запускаем в докере
minikube start --vm-driver=docker

# Проверяем статус
minikube status
kubectl get po -A

kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080

kubectl get services hello-minikube

# Проверяем в баузере
minikube service hello-minikube
```

### Kubeadm
```txt
Последовательность настройки:

Устанавливается среда исполнения контейнеров, например containerd
Устанавливается kubeadm для настройки узлов
Инициализация мастер узла
Настройка Pod Network
Подключение рабочих узлов
```