# 部署 KubeSphere

## 准备工作

### 安装 helm

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

### 查看 master 节点是否有 Taint

```bash
$ kubectl describe node master | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
```

### 去掉 master 节点的 Taint

```bash
$ kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
```

## 安装 默认存储（Dev 环境 这里使用OpenEBS）

1. 创建 OpenEBS 的 namespace，OpenEBS 相关资源将创建在这个 namespace 下：

```bash
kubectl create ns openebs
```

1. 安装 OpenEBS，以下列出两种方法，可参考其中任意一种进行创建：

A. 若集群已安装了 Helm，可通过 Helm 命令来安装 OpenEBS：

```plain
helm init
helm install --namespace openebs --name openebs stable/openebs --version 1.5.0
```

B. 除此之外 还可以通过 kubectl 命令安装：

```bash
$ kubectl apply -f https://openebs.github.io/charts/openebs-operator-1.5.0.yaml
```

1. 安装 OpenEBS 后将自动创建 4 个 StorageClass，查看创建 StorageClass：

```bash
$ kubectl get sc
NAME                        PROVISIONER                                                AGE
openebs-device              openebs.io/local                                           10h
openebs-hostpath            openebs.io/local                                           10h
openebs-jiva-default        openebs.io/provisioner-iscsi                               10h
openebs-snapshot-promoter   volumesnapshot.external-storage.k8s.io/snapshot-promoter   10h
```

2. 如下将`openebs-hostpath`设置为 默认的 StorageClass：

```bash
$ kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/openebs-hostpath patched
```

3. 至此，OpenEBS 的 LocalPV 已作为默认的存储类型创建成功。可以通过命令`kubectl get pod -n openebs`来查看 OpenEBS 相关 Pod 的状态，若 Pod 的状态都是 running，则说明存储安装成功。

4. 我们可以在安装完 OpenEBS 和 KubeSphere 后，可以将 master 节点 Taint 加上，避免业务相关的工作负载调度到 master 节点抢占 master 资源

```bash
kubectl taint nodes master node-role.kubernetes.io/master=:NoSchedule
```

## yaml


```bash
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.0.0/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.0.0/cluster-configuration.yaml
```

检查安装日志：

```bash
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

# 

## 本地部署

1、安装 minikube

2、输入命令初始化集群

```bash
minikube start --cpus=4 --memory=8096mb --kubernetes-version=v1.18.8 --driver=virtualbox --image-mirror-country cn --registry-mirror="https://docker.mirrors.ustc.edu.cn"

```

3、初始化 kubesphere


```bash
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.0.0/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.0.0/cluster-configuration.yaml
# 检查安装日志
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
# 使用 kubectl get pod --all-namespaces 查看所有 Pod 是否在 KubeSphere 的相关命名空间中正常运行。如果是，请通过以下命令检查控制台的端口（默认为 30880）：
kubectl get svc/ks-console -n kubesphere-system
# 确保在安全组中打开了端口 30880，并通过 NodePort（IP：30880）使用默认帐户和密码（admin/P@88w0rd）访问 Web 控制台。
# 查看jwt
kubectl -n kubesphere-system get cm kubesphere-config -o yaml | grep -v "apiVersion" | grep jwtSecret
```