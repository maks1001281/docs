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