## docker

打包
> docker build -t kubia .

运行
> docker run --name kubia-test -p 8080:8080 -d kubia

访问
> curl http://localhost:8080

查看细节
> docker inspect kubia-test

进入容器
> docker exec -it kubia-test bash

列出容器中进程
> ps aux

打标签(lever0066替换成自己的)
> docker tag kubia lever0066/kubia

推送到docker-hub
> docker push lever0066/kubia

停止容器
> docker stop kubia-test

删除容器
> docker rm kubia-test

## k8s

使用kubectl列出集群节点
> kubectl get nodes

部署nodejs应用
> kubectl run kubia --image=lever0066/kubia --port=8080 --generator=run-pod/v1

列出pods
> kubectl get pods

查看详细（如有错也可以查看）
> kubectl describe pod 

删除rc
> kubectl delete rc kubia 

通过创建 LoadBalancer 类型的服务，将创建一个外部的负载均衡，可以通过负载均衡的公共IP访问pod。
> kubectl expose rc kubia --type=LoadBalancer --name kubia-http

查看下刚才暴漏的服务
> kubectl get services 

查看下
> kubectl get rc 

访问一下
> curl localhost:8080

扩容一下，变成3个
> kubectl scale rc kubia --replicas=3

显示pod的IP和所运行的节点
> kubectl get pods -o wide

查看某一个pod的细节
> kubectl describe pod kubia-6cwt6

查看现有pod的yaml描述文件
> kubectl get po kubia-6cwt6 -o yaml

通过yaml文件创建pod
> kubectl create -f .\kubia-manual.yaml

查看日志
> kubectl logs kubia-manual 

创建带标签的pod