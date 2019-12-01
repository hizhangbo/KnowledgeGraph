# Kubernetes
[Kubernetes部署](https://yq.aliyun.com/articles/682810 "利用 Kubeadm部署 Kubernetes 1.13.1 集群实践录")  
[Dashboard安装](https://andrewpqc.github.io/2018/04/24/setup-k8s-dashboard-on-cluster "Dashboard生成SSL安全证书")

1. VMware安装一台centos7，并安装docker  
2. 拷贝多份虚拟机，并为每一台修改 host  
```shell
hostnamectl --static set-hostname  kuber-master  
hostnamectl --static set-hostname  kuber-slave1  
hostnamectl --static set-hostname  kuber-slave2  
```
3. 修改ifcfg-env hosts 文件，并重启网络 service restart network
4. 硬件限制  
每台机器2 GB或更多的RAM（任何更少的内存将为您的应用留下很小的空间）
2个CPU或更多  
[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2  
kubeadm init --kubernetes-version=v1.13.1 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU  
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster  
5. no set 1 error  
将命令echo "1" > /proc/sys/net/ipv4/ip_forward 写入脚本/etc/rc.d/rc.local  
```shell
echo "1" > /proc/sys/net/ipv4/ip_forward  
echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables
```
6. 查看占用端口进程  
```shell
yum -y install net-tools  
netstat -tunlp|grep port  
```
7. 安装
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version

https://kubernetes.io/docs/tasks/tools/install-kubectl/
```
8. 查看状态(STATUS:ContainerCreating)  
```shell
kubectl get pods --all-namespaces -o wide  
```
9. 容器详细描述  
```shell
kubectl describe node server0  
```
10. 对于swap的限制  
```shell
# 重启后失效
swapoff -a   
# 要永久禁掉swap分区，打开如下文件注释掉swap那一行 
vim /etc/fstab
```
11. 删除pod  
```shell
kubectl delete pod NAME --grace-period=0 --force  
```
12. Dashboard访问密钥
```shell
kubectl create secret generic kubernetes-dashboard-certs --from-file=/home/share/certs -n kube-system  
```
13. 生成token  
```shell
kubectl describe secret/$(kubectl get secret -nkube-system |grep admin|awk '{print $1}') -nkube-system  
```
14. kubectl命令自动补全  
```shell
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)  
```
15. 删除deployment  
```shell
kubectl get deployments  
kubectl delete deployments [NameOfDeployment]  
kubectl get rc
kubectl delete deployments [NameOfrc]  
```
16. 登录aliyun 镜像仓库  
```shell
docker login registry.cn-shenzhen.aliyuncs.com/hizhangbo/test -u=[account id]  
```
17. kompose安装  
```shell
# Linux
curl -L https://github.com/kubernetes/kompose/releases/download/v1.18.0/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
# autocomplete
# Bash (add to .bashrc for persistence)
source <(kompose completion bash)
```
18. 本地调试  
*[kubernetes官网](https://kubernetes.io/docs/tasks/debug-application-cluster/local-debugging/ "kubernetes官网")*  
*[telepresence官网](https://www.telepresence.io/ "telepresence官网")*  
*[kubeadm安装文档](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/ "kubeadm安装文档")*
