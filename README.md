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

# Minikube - предварительно настроенный одноузловой кластер kubernetes, который является и мастером и воркером

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

# Удаление minikube
minikube delete
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

### Абстракции kubernetes
#### PODs
```txt
В идеале, один под - один контейнер
Можно в поде хранить несколько контейнеров, но для масштабирования приложения, лучше чтобы был контейнер другого типа
```
```bash
# добавляем pod в котором будет контейнер из образа nginx
kubectl run nginx --image nginx
# список подов и статус
kubectl get pods -owide
# посмотреть информацию о поде
kubectl describe pod nginx
```

#### YAML
```txt
Массив:
  - Элемент1
  - Элемент 2

Список ключ-значений:
  ключ1: значение1
  ключ2: значение2
```

### PODs + YAML
```txt
Манифест кубернетес всегда содержит 4 верхнеуровневых поля: apiVersion, king, metadata, spec
kind - тип создаваемого обьекта: Pod, Service, ReplicaSet, Deployment
metadata - данные об объекте
```
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx

# Теперь можно создавать под на основе файла конфигурации
kubectl create -f pod-definition.yml
# Если мы решили поменять конфигурацию (меняем файл и применяем настройки)
kubectl apply -f pod-definition.yml
# Или так
kubectl edit pod myapp-pod

kubectl get pods
kubectl describe pod myapp-pod
```

### ReplicaSet
```txt
Replication Controller - старая технология, сейчас используется ReplicaSet
С помощью selector, ReplicaSet может управлять подами, не созданными с помощью RS
Кол-во реплик можно увеличить с помощью команды kubectl scale --replicas=6 -f rs-definition.yml
```
```yaml
# Replication Controller
# rc-definition.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
replicas: 3
```
```bash
kubectl create -f rc-definition.yml
kubectl get replicationcontroller
kubectl get pods
```
```yaml
# rs-definition.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchlabels:
      type: front-end
```
```bash
kubectl create -f rs-definition.yml
kubectl get replicaset
kubectl get pods
kubectl scale --replicas=6 -f rs-definition.yml
kubectl get pods
```

### Сетевое взаимодействие
```txt
Каждый pod получает свой адрес во внутреннней сети.

Kubernetes не настраивает, а ожидает, что мы настроим сеть.
Нужно выбрать плагин для настройки сети.
```

### Service
```bash
Service - служба NodePort прослушивает порт на ноде и перенаправляет запросы на под
ClusterIP - создает виртуальный IP внутри кластера, чтобы обеспечить связь между службами
LoadBalancer - балансировщик нагрузки

NodePort 30080 -> Service Port 80 -> Target POD Port 80
```

```yml
# service-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort
  ports:
    - targetPort: 80 # Если не указан, то считается, что равен порту службы (port)
      port: 80 # Обязательное поле
      nodePort: 30080 # Если не указан, выделяется автоматически
  selector:
    app: myapp
    type: front-end # определяем поды, к которым создать связь
```

```bash
cubectl create -f service-definition.yml
cubectl get services
```