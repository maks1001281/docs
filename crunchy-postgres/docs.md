<details>
<summary>Резервное копирование и востановление</summary>
Для выполнения полного ручного резервного копирования в spec должна присутствовать конфигурация:

```
spec:
  backups:
    pgbackrest:
      manual:
        repoName: repo1
        options:
         - --type=full
```
Резервное копирование выполняется командой:
```
kubectl annotate -n crunchy-postgres postgrescluster mypg postgres-operator.crunchydata.com pgbackrest-backup="$(date)"
```
Для версии postgres-operator-examples-8-5.1.2-0 копирование выполняется командой:
```
kubectl annotate -n crunchy-postgres postgrescluster clustername postgres-operator.crunchydata.com/pgbackrest-backup="$( date '+%F_%H:%M:%S' )"
```
Ключ `--overwrite` перезапишет последний актуальный бэкап.

Для востановления только что созданного бэкапа в spec должна присутствовать конфигурация:

```
spec:
  dataSource:
    postgresCluster:
      clusterName: hippo
      repoName: repo1
      options:
      - --type=time
      - --target="2024-01-12 14:15:11-04"
```
Опция `- --target` указывает на какое время необходимо востановиться, цыфры после времени указывают часовой пояс по UTC, в нашем случае новосибирское время указывается вот так `2024-01-12 14:15:11+07`

Востановление выполняется командой:

`kubectl annotate -n crunchy-postgres postgrescluster mypg --overwrite postgres-operator.crunchydata.com pgbackrest-restore=id1`

Где `restore=id1` номер резервного востановления, те создать еще одно такое не получится, что бы пересоздать нужно изменить номер`id1`

Ключ `--overwrite` перезапишет текущий рабочий кластер из бэкапа.

Для подробной информации стоит посетить оф [документацию](https://access.crunchydata.com/documentation/postgres-operator/latest/tutorials/backups-disaster-recovery/disaster-recovery)

</details>