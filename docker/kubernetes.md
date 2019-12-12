**安装部署**
  1. kubeadm 环境准备
```shell
sysctemctl stop firewalld && sysctemctl disable firewalld
yum install -y docker
systemctl enable docker && systemctl start docker

# 重启后失效
swapoff -a   
# 要永久禁掉swap分区，打开如下文件注释掉swap那一行 
vim /etc/fstab

# RHEL / CentOS 7上的一些用戶報告了由於iptables被繞過而導致流量路由不正確的問題。您應該確保 net.bridge.bridge-nf-call-iptables在sysctl配置中設置為1 
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

# 正式开始安装
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
# 切换阿里环境
# gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
# 安全增强型 Linux(Security-Enhanced Linux)简称 SELinux 主要作用就是最大限度地减小系统中服务进程可访问的资源(最小权限原则)
/usr/sbin/sestatus -v # 查看SELinux状态
# 关闭 SELinux 权限限制
# 第一种方式临时关闭，不需要重启机器
setenforce 0
# 第二种方式修改配置文件
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```
  2. 配置ssh免密码访问
```shell
-- 修改hostname
hostnamectl --static set-hostname  kuber-master

ssh -o "StrictHostKeyChecking no" root@123456
ssh-keygen
ssh-copy-id -i ~/.ssh/id_rsa.pub root@kube1
ssh-copy-id -i ~/.ssh/id_rsa.pub root@kube2
```
  3. 初始化master节点
```shell
# 下载镜像
kubeadm config images list
kubeadm config images pull

# k8s.gcr.io/kube-apiserver:v1.15.3
# k8s.gcr.io/kube-controller-manager:v1.15.3
# k8s.gcr.io/kube-scheduler:v1.15.3
# k8s.gcr.io/kube-proxy:v1.15.3
# k8s.gcr.io/pause:3.1
# k8s.gcr.io/etcd:3.3.10
# k8s.gcr.io/coredns:1.3.1

kubeadm init --kubernetes-version=v1.15.3 --pod-network-cidr=10.244.0.0/16 

kubeadm join 192.168.127.70:6443 --token t0xrfm.n1cpx8k1g84e7u8u \
    --discovery-token-ca-cert-hash sha256:49772aa397f90e5d06d0c02e080156cdd740930d0fa3f371f2dcd87790dce568

# 配置普通用户可以运行 kubectl 命令
# root 账号执行
export KUBECONFIG=/etc/kubernetes/admin.conf
# 普通账号执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
  4. 安装pod網絡加載項
```shell
# For flannel to work correctly, you must pass --pod-network-cidr=10.244.0.0/16 to kubeadm init
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml

# For Calico to work correctly, you need to pass --pod-network-cidr=192.168.0.0/16 to kubeadm init
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```
  5. 安装dashboard
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta3/aio/deploy/recommended.yaml
kubectl proxy --address 0.0.0.0 --accept-hosts '.*'
kubectl proxy
```
**常用命令**
- kubeadm
  1. kubeadm token create # 令牌在24小時後過期,
  2. kubeadm token list
- kubectl
  1. kubectl version # 查看Kubernetes集群和客户端版本
  2. kubectl api-versions # 查看api版本 group/version
  3. kubectl get pods --all-namespaces # 获取所有 pod
  4. kubectl get nodes # 获取所有集群节点
  5. kubectl cluster-info # 获取集群信息
  6. 从集群中移除节点
     1. kubectl drain <node name> --delete-local-data --force --ignore-daemonsets # 主节点上运行
     2. kubectl delete node <node name> # 主节点上运行
     3. kubeadm reset # 在要刪除的節點上，重置所有kubeadm安裝狀態
  7. kubectl describe nodes kube0 # 获取单个节点描述
  8. kubectl describe pods etcd-kube0 -n kube-system # 获取单个pod描述
  9. kubectl get deployment coredns -n kube-system # 查看部署的pod集群状态
  10. kubectl get deployment coredns -n kube-system -o yaml # 查看部署时的yaml描述
  11. kubectl edit service kubernetes-dashboard -n kubernetes-dashboard # 编辑部署时的yaml描述
  12. kubectl get service kubernetes-dashboard -n kubernetes-dashboard
  13. kubectl get pod kubernetes-dashboard-7d8b9cc8d-fz5qz -n kubernetes-dashboard -o wide
  14. kubectl get pod kubernetes-dashboard-7d8b9cc8d-fz5qz -n kubernetes-dashboard -o yaml
  15. kubectl get svc --all-namespaces # service
  16. kubectl get rc --all-namespaces # ReplicaController
  17. kubectl get rs --all-namespaces # ReplicaSet
  18. kubectl expose pods kubernetes-dashboard-7d8b9cc8d-fz5qz --type=NodePort -n kubernetes-dashboard # 暴露服务端口到外部
  19. kubectl delete service kubernetes-dashboard-7d8b9cc8d-fz5qz -n kubernetes-dashboard # 删除创建的服务
  20. kubectl get node --show-labels # 显示标签
  21. kubectl label node kube1 label:value # 设置标签
  22. kubectl get namespaces
  23. kubectl logs dashboard-metrics-scraper-fb986f88d-2266s -n kubernetes-dashboard # 打印日志
  24. kubectl exec 
  25. kubectl patch svc kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}' -n kubernetes-dashboard # 以打补丁方式修改dasboard的访问方式
  26. kubectl get secret -n kubernetes-dashboard # 获取身份认证服务
  27. kubectl describe secret kubernetes-dashboard-token-wl8wl -n kubernetes-dashboard # 获取登录token
  28. kubectl get sa --all-namespaces # 获取serviceAccount
  29. kubectl get role --all-namespaces # 获取集群角色
  30. kubectl get clusterrolebinding # 获取集群角色绑定
  31. kubectl describe clusterrolebinding cluster-admin # 角色绑定详情描述
  32. kubectl describe role kubernetes-dashboard -n kubernetes-dashboard # 获取角色详情
  33. kubectl describe clusterrole cluster-admin -n kube-system # 获取集群角色详情
  34. kubectl create clusterrolebinding kubernetes-dashboard-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:kubernetes-dashboard # 绑定serviceaccount和clusterrole, namespace:name
  35. kubectl describe secret kubernetes-dashboard-token-wl8wl -n kubernetes-dashboard # 获取集群权限token
  
**问题解决**
  1. coredns 启动失败
```shell
kubectl get deployment coredns -n kube-system -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -

似乎有3種不同的解決方法：

升級到更新版本的docker，例如17.03，k8s目前推薦的版本
或者從部署的pod規範中刪除allowPrivilegeEscalation = false
或者禁用SELinux
```

# 其它资料
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
