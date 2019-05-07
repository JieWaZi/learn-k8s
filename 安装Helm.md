#### 常规安装方式
1. 下载对应包：
```
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.12.3-linux-amd64.tar.gz

$ tar -xzvf helm-v2.12.3-linux-amd64.tar.gz

$ mv linux-amd64/helm /usr/local/bin/helm
```

2. 初始化 helm并安装Tiller
```
$ helm init --upgrade -i jmgao1983/tiller:v2.12.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```
3. 查看tiller运行情况并创建相关权限
```
$ kubectl -n kube-system get pods|grep tiller

$ kubectl create serviceaccount --namespace kube-system tiller

$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```
4. 查看版本
```
$ helm version
Client: &version.Version{SemVer:"v2.12.3", GitCommit:"eecf22f77df5f65c823aacd2dbd30ae6c65f186e", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.12.3", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
```

#### 直接使用ansible的脚本
```
ansible-playbook /etc/ansible/roles/helm/helm.yml
```
该方法使用了TLS认证。
