<details>
<summary>Отключение проверки https для хранилища образов </summary>

Создаем файл по пути `/etc/docker/daemon.json` с содержимым 

```
{ "insecure-registries":["192.168.1.14:8082"] }
```

</details>