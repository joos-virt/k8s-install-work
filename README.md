# k8s-install-work
Установка, запуск приложения

В лоб не стартануло на 11 дебиане

**Ставим докер или любую другую контейнерную среду.**

https://docs.docker.com/engine/install/debian/
```
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```
```
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
```
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
```
sudo docker run hello-world
```

**Установка kubeadm, kubelet, kubectl**

Устанавливаем зависимости для apt:
```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl
```
Настраиваем репозиторий:
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```
Устанавливаем пакеты:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
**Создание кластера**

Инициализируем первую мастер ноду:
```
kubeadm init
```
[WARNING Swap]: swap is enabled
```
swapoff -a - отключаем swap до перезагрузки
```
[ERROR CRI]: container runtime is not running: output:
```
find / -name *.toml - /etc/containerd/config.toml - находим
rm /etc/containerd/config.toml - и удаляем файл
systemctl restart containerd - перезапускаем
```
Если всё прошло успешно, то в консоль выведется команда для присоединения нод к кластеру, выполняем её:
```
kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
Для постоянной работы после перезагрузки:
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf - выбираем для тестов

kubectl get pods --all-namespaces - проверяем что запустилось
kubectl describe pod/coredns-5d78c9869d-h74sx -n kube-system
```
Устанавливаем CNI, например weave:
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
```
kubectl get nodes - Отображает ноды
```
**Helm**
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 |bash
```
Helm можно использовать для установки чартов из публичныхрепозиториев:
```
helm repo add stable https://charts.helm.sh/stable
helm search repo wordpress
helm install my-wordpress stable/wordpress - не поднимится, так как нужны Persistent Volumes - https://kubernetes.io/docs/concepts/storage/persistent-volumes/
```
И для создания, и деплоя своих собственных чартов:
```
helm create mychart
```
Структура
```
mychart/
Chart.yaml
values.yaml
templates/
  configmap.yaml
```
Переходим в templates и удаляем содержимое
```
rm -rf ./*
```
И создаем

configmap.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  test: {{ .Values.replicas | quote }}
```
Установим наш чарт
```
helm install mycahart ./mychart
```
Проверим его манифест
```
helm get manifest mycahart
```
Удалим чарт
```
helm uninstall mycahart
```



**Rotoro**
```
kubectl run nginx --image=nginx - создаем
kubectl get pods - смотрим
kubectl describe pod nginx - подробная информация
kubectl get pods -o wide - показывает дополнительные поля
kubectl delete pod nginx
```
pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    type: frontend
spec:
  containers:
  - name: nginx
    image: nginx
```
```
kubectl create -f pod.yml - создаем
kubectl get pods - смотрим
kubectl describe pod nginx - подробная информация
```
replicasets.yml
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
spec:
  template:
    metadata:
      name: nginx2
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      app: myapp
```
```
kubectl create -f replicasets.yml - создаем
kubectl get rs - смотрим replicasets
kubectl get po - смотрим поды
kubectl delete po myapp-replicaset-28w4l - удаляем под
kubectl get po - проверяем что автоматом поднялся еще один
kubectl edit rs myapp-replicaset - редактируем rs через vi
kubectl scale rs myapp-replicaset --replicas=2 - или редактор реплик на лету
kubectl delete rs myapp-replicaset
```
