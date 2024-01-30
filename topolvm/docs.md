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