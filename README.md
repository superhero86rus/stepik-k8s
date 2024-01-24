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
```