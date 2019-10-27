---
title: 解决kubespray 安装 k8s 集群的Check_certs | Set 'sync_certs' to true'  错误
permalink: kubespray-k8s-error-sync_certs
id: 5d5e865ce9326100bc7569ba
updated: '2017-11-28T02:41:58.000Z'
date: 2017-11-28 10:41:58
---

在 kubespray 的 issue 上看到的<a href="" target="_blank">Failed Kargo deploy at 'etcd : Check_certs | Set 'sync_certs' to true'</a>
## 解决方法
```
yum -y install python-pip
pip install https://pypi.python.org/packages/2.7/J/Jinja2/Jinja2-2.8-py2.py3-none-any.whl
```