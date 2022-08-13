# Ingress部署

以版本v0.30.0为例

## 下载yaml文件

```bash
$wget https://github.com/kubernetes/ingress-nginx/blob/nginx-0.30.0/deploy/static/mandatory.yaml
$wget https://github.com/kubernetes/ingress-nginx/blob/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml

# 也可以从我的github上下载, 注意我仓库里的yaml文件叫deplay.yaml
https://github.com/seepre/kubernetes-kubeadm/tree/main/ingress
```

## 然后将下载的yaml文件apply起来

```bash
$ kubectl apply -f mandatory.yaml
$ kubectl apply -f service-nodeport.yaml
```

## 查看service和ingress信息

```bash
$ kubectl get svc -n ingress-nginx
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.103.255.100   <none>        80:32522/TCP,443:30946/TCP   24m
```

然后我们使用http://47.97.40.26:32522/ 就可以访问了

