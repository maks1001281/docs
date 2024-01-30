<details>
<summary>Проблема зависания PVC в статусе pending на версии тополя 0.10.5</summary>

Для того, чтобы ```topolvm``` создавал ```pvc``` нужно сделать:

- либо на каждый ```namespace``` навешивать метку ```kubectl label namespace <your namespace> topolvm.cybozu.com/webhook=ignore``` где будут создаваться ```pvc```
- либо подправить конфигурацию ```webhook``` в ```values.yaml```, указав:

```yaml
 webhook:
  podMutatingWebhook:
    enabled: false
  pvcMutatingWebhook:
    enabled: true
```

</details>

<details>
<summary>Проблема поднятия подов topolvm-controller при первом deploy</summary>

Проблема:
```
 Warning  FailedMount  2m22s (x3 over 6m54s)  kubelet            Unable to attach or mount volumes: unmounted volumes=[certs], unattached volumes=[socket-dir certs kube-api-access-m64pd]: timed out waiting for the condition
  Warning  FailedMount  56s (x13 over 11m)     kubelet            MountVolume.SetUp failed for volume "certs" : secret "topolvm-mutatingwebhook" not found
  Warning  FailedMount  4s (x2 over 9m9s)      kubelet            Unable to attach or mount volumes: unmounted volumes=[certs], unattached volumes=[kube-api-access-m64pd socket-dir certs]: timed out waiting for the condition
```

При первом деплое topolvm ходит в ```cert-manager```  что бы запросить сертификаты и записать их в secret ```topolvm-mutatingwebhook```, ```topolvm``` работает только с версией ```cert-manager v1.7.0 ```
при установке более новой версии у меня возникала эта ошибка, в ```dependencies``` Chart.yaml в старых и новых версиях ```topolmv```
требование к ```cert-manager``` v1.7.0

</details>