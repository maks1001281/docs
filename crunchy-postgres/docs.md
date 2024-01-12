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
</details>