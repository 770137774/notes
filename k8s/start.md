##1、运行Etcd
```
docker run --net=host -d gcr.io/google_containers/etcd:2.0.12 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir=/var/etcd/data
#容器中的app无法访问到宿主机，因为两者不在一个网络内。最简单的方式是在启动docker时增加–net=host选项，这样容器就和宿主机共用网络，容器中的app也就能访问了  
--net=host    
#后台运行容器，并返回容器ID
-d  
```

kubeadm安装新版本的Kubernetes过程中，需要从k8s.grc.io仓库中拉取所需镜像文件，但由于TheGreatWall导致无法正常拉取，可以使用以下方式绕过此问题。

docker.io仓库对google的容器做了镜像，可以通过下列命令下拉取相关镜像：

docker pull mirrorgooglecontainers/kube-apiserver:v1.13.2

docker pull mirrorgooglecontainers/kube-controller-manager:v1.13.2

docker pull mirrorgooglecontainers/kube-scheduler:v1.13.2

docker pull mirrorgooglecontainers/kube-proxy:v1.13.2

docker pull mirrorgooglecontainers/pause:3.1

docker pull mirrorgooglecontainers/etcd:3.2.24

docker pull coredns/coredns:1.2.6

通过docker tag命令来修改镜像的标签：

docker tag docker.io/mirrorgooglecontainers/kube-apiserver:v1.13.2 k8s.gcr.io/kube-apiserver:v1.13.2

docker tag docker.io/mirrorgooglecontainers/kube-controller-manager:v1.13.2 k8s.gcr.io/kube-controller-manager:v1.13.2

docker tag docker.io/mirrorgooglecontainers/kube-scheduler:v1.13.2 k8s.gcr.io/kube-scheduler:v1.13.2

docker tag docker.io/mirrorgooglecontainers/kube-proxy:v1.13.2 k8s.gcr.io/kube-proxy:v1.13.2

docker tag docker.io/mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1

docker tag docker.io/mirrorgooglecontainers/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24

docker tag docker.io/coredns/coredns:1.2.6 k8s.gcr.io/coredns:1.2.6
##2、启动master
```
docker run \
    --volume=/:/rootfs:ro \
    --volume=/sys:/sys:ro \
    --volume=/dev:/dev \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --volume=/var/lib/kubelet/:/var/lib/kubelet:rw \
    --volume=/var/run:/var/run:rw \
    --net=host \
    --pid=host \
    --privileged=true \
    -d \
    gcr.io/google_containers/hyperkube:v1.0.1 \
    /hyperkube kubelet --containerized --hostname-override=&quot;127.0.0.1&quot; --address=&quot;0.0.0.0&quot; --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests
```
>参考文档
- https://www.kubernetes.org.cn/doc-5
- https://blog.csdn.net/jinguangliu/article/details/82792617