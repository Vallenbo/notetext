[containerd downloads](https://containerd.io/downloads/)

# containerd命令

```sh
# 查看镜像
ctr image list
或者
crictl images
 
# 拉取镜像
ctr i pull --all-platforms registry.xxxxx/pause:3.2
或者
ctr i pull --user user:passwd --all-platforms registry.xxx.xx/pause:3.2
或者
crictl pull --creds user:passwd registry.xxx.xx/pause:3.2

# 镜像打tag
镜像标记tag
ctr -n k8s.io i tag registry.xxxxx/pause:3.2 k8s.gcr.io/pause:3.2
或者
ctr -n k8s.io i tag --force registry.xxxxx/pause:3.2 k8s.gcr.io/pause:3.2

# 删除镜像tag
ctr -n k8s.io i rm registry.xxxxx/pause:3.2

# 推送镜像
ctr images push --user user:passwd registry.xxxxx/pause:3.2
```





