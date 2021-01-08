# telepresence 替换 k8s 内 service

## 替换 kubesphere apiserver 本地端口为 9091 远程端口为 9090

```shell
telepresence --namespace kubesphere-system --swap-deployment ks-apiserver --also-proxy redis.kubesphere-system.svc --also-proxy openldap.kubesphere-system.svc --also-proxy prometheus-operated.kubesphere-monitoring-system.svc --expose 9091:9090
```
## 启动 apiserver

```shell
./ks-apiserver--insecure-port9091--kubeconfig~/.kube/config-v8
```
