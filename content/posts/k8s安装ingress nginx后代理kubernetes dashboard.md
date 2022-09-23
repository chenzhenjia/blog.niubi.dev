---
title: k8s 安装ingress-nginx后代理kubernetes-dashboard
aliases: ["/k8s-an-zhuang-ingress-nginxhou-dai-li-kubernetes-dashboard/"]
date: 2018-04-30T19:06:43+08:00
---

# 准备
1. k8s 集群，版本为1.9.5
2. kubernetes-dashboard 为 v1.8.3

## 安装 kubernetes-dashboard
1. 直接使用官方的命令安装
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
    ```
1. 暴露端口，以便访问，用完之后改回
    ```bash
    kubectl -n kube-system edit service kubernetes-dashboard
    ```
    按 i 进入插入模式
    修改 type 值 ClusterIP 为NodePort
    然后按 esc ，再按 x 回车保存
1. 查看端口
    ```bash
    kubectl -n kube-system get service kubernetes-dashboard
    ```
    ```bash
    NAME                   CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes-dashboard   10.100.124.90   <nodes>       443:31707/TCP   21h
    ```
    其中 31707就是外部可以访问的 ip
1. 创建管理员
    // admin-role.yaml
    ```yaml
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: admin
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
    roleRef:
      kind: ClusterRole
      name: cluster-admin
      apiGroup: rbac.authorization.k8s.io
    subjects:
    - kind: ServiceAccount
      name: admin
      namespace: kube-system
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: admin
      namespace: kube-system
      labels:
        kubernetes.io/cluster-service: "true"
        addonmanager.kubernetes.io/mode: Reconcile
    ```
    
    ```bash
    kubectl create -f admin-role.yaml
    ```
    ```bash
    kubectl -n kube-system get secret|grep admin-token
    ```
    // 显示
    ```bash
    admin-token-jlwhj                                kubernetes.io/service-account-token   3         3h
    ```
    ```bash
    kubectl -n kube-system describe secret admin-token-jlwhj
    ```
    ```text
    Name:         admin-token-jlwhj
    Namespace:    kube-system
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name=admin
                  kubernetes.io/service-account.uid=35dda908-4c49-11e8-ab43-ae622023bc2f

    Type:  kubernetes.io/service-account-token

    Data
    ====
    ca.crt:     1090 bytes
    namespace:  11 bytes
    token:      xxxx
    ```
    // 复制好 token 里面的值，登录会用到
3. 进入dashboard 管理页面
    // 浏览器输入 https://ip:31707 进入页面,点右上角，登录,然后点击令牌，把 token 复制进去，然后登录

## 安装 ingress-nginx
1. 使用 ingress-nginx 官方命令安装
    ```bash
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml \
    | kubectl apply -f -
    ```
    ```bash
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml \
        | kubectl apply -f -
    ```    
    ```bash
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml \
        | kubectl apply -f -
    ```    
    ```bash
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml \
        | kubectl apply -f -
    ```    
    ```bash
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml \
        | kubectl apply -f -
    ```    
    ```bash    
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/rbac.yaml \
        | kubectl apply -f -
    ```    
    ```bash    
    curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/without-rbac.yaml \
    | kubectl apply -f -    
    ```
1. 进入界面修改 ingress-nginx 的 hostNetwork 模式
    1. 点击 nginx-ingress-controller 然后点编辑
    2. 找到 spec -> template -> spec 的节点
    3. 添加 "hostNetwork":true
    4. 点击更新
    5. 测试 80 端口是否开放，修改完成

## 代理 kubernetes-dashboard
1. 创建 代理配置
    由于我的kubernetes-dashboard是在 kube-system 命名空间下，所以需要在 kube-system 命名空间下创建 ingress 的配置
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      annotations:
        nginx.ingress.kubernetes.io/secure-backends: "true" 
      name: ingress-proxy
      namespace: kube-system
      force-ssl-redirect: true
      ssl-redirect: true
      use-proxy-protocol: true
    spec:
      tls:
      - hosts:
        - k8s.xxx.com
        secretName: ingress-ai5suoai-com-secret
      rules:
      - host: k8s.xxx.com
        http:
          paths:
          - backend:
              serviceName: kubernetes-dashboard
              servicePort: 8443
    ```
    *nginx.ingress.kubernetes.io/secure-backends: "true" # 注意这行，没有这个 kubernetes-dashboard 会报 TLS handshake error: first record does not look like a TLS handshake 异常*
