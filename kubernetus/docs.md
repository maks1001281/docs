<details>
<summary>Копирование файлов с хоста в pod</summary>

`kubectl cp filename namespace/podname:/path_pod`

Пример команды:

`kubectl cp demo-small-20170815.sql default/pgback:/tmp`

</details>

<details>
<summary>Быстрое создание pod</summary>

`kubectl run -it podname --image=image -- bash`

Пример команды:

`kubectl run -it pgto --image=registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-16.1-0 -- bash`

</details>

<details>
<summary>Установка нужной версии kubeadm kubectl kubelet</summary>

Используем официальный старый репо который больше не поддерживается:

`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`

`sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"`

`sudo apt-get update`

`sudo apt-get install -qy kubeadm=1.23.5-00 kubectl=1.23.5-00 kubelet=1.23.5-00`

</details>

<details>
<summary>Подключение control plane node в кластер с помощью kubeadm</summary>

Если при подключении master mode возникает ошибка:

```
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
error execution phase preflight: 
One or more conditions for hosting a new control plane instance is not satisfied.

unable to add a new control plane instance to a cluster that doesn't have a stable controlPlaneEndpoint address

Please ensure that:
* The cluster has a stable controlPlaneEndpoint address.
* The certificates that must be shared among control plane instances are provided.


To see the stack trace of this error execute with --v=5 or higher
```

То у нас проблема с сертификатами и стадии инициализации control plane нужно пройти в ручную:

- Создаем папки на новой ноде `mkdir /home/$USER/pki` ,  `mkdir /home/$USER/pki/etcd`
- Копируем сертификаты и конфиг с рабочей мастер ноды на новую
`scp /etc/kubernetes/pki/ca.{key,crt} maks@192.168.1.22:/home/maks/pki/`
`scp /etc/kubernetes/pki/etcd/ca.{key,crt} maks@192.168.1.22:/home/maks/pki/etcd`
`scp /etc/kubernetes/admin.conf maks@192.168.1.22:/home/maks/`
- Копируем сертификаты и конфиг в папки на новой ноде, если нету создаем
`/etc/kubernetes/pki/`
`/etc/kubernetes/pki/etcd/`
`/etc/kubernetes/admin.conf`
- Делаем `export KUBECONFIG=/etc/kubernetes/admin.conf`
- Копируем конфиг кластера с рабочей мастер ноды на новую ``sudo scp /var/lib/kubelet/config.yaml maks@192.168.1.22:/home/maks/``
- Меняем там параметры на свои в полях, сохраняем:
```
advertiseAddress:
token:
node-ip:
name: 
```
Параметр токен можно узнать командой на рабочей ноде `kubeadm token list` если нет то генерируем новый `kubeadm token create`
- Заходим под пользователем **sudo** проверяем доступ к кластеру `kubectl get po -A`
- Инициализируем фазу сертификатов
`kubeadm init phase certs all`
- Запускаем kubelet с нашим конфигом кластера
`kubeadm join phase kubelet-start 192.168.1.21:6443 --config conf.yaml`
- Инициализируем фазу control-plane-prepare
`kubeadm join phase control-plane-prepare kubeconfig 192.168.1.21:6443 --config conf.yaml`
`kubeadm join phase control-plane-prepare control-plane --config conf.yaml`
- Инициализируем фазу etcd
`kubeadm join phase control-plane-join etcd --config conf.yaml`
- Проверяем подключение

</details>