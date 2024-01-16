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