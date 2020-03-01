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

#### POD

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

查看之前为什么被终止的log
> kubectl logs  kubia-liveness --previous

#### 万能的标签

创建带标签的pod
> kubectl create -f .\kubia-manual-with-labels.yaml

查看pod的标签
> kubectl get po --show-labels

查看某个标签的pod(包含未打标签的pod)
> kubectl get po -L env

查看某个标签的pod(只包含指定标签的pod)
> kubectl get po -L env

查看某个标签的pod(标签=指定值的pod)
> kubectl get po -l env=debug

列出没有env标签的pod(确保使用单引号来圈引！env，这样bash shell才不会解释感叹号)
> kubectl get po -l '!env'

标签其他操作：
- creation_method!=manual 选择带有creation_method标签，并且值不等于manual的pod
- env in（prod,devel）选择带有env标签且值为prod或devel的pod
- env notin（prod,devel）选择带有env标签，但其值不是prod或devel的pod
- 在包含多个逗号分隔的情况下，可以在标签选择器中同时使用多个条件，此时资源需要全部匹配才算成功匹配了选择器。（如app=pc,rel=beta）


通过命令给某个pod添加标签
> kubectl label po kubia-manual creation_method=manual

把env=prod改成debug
> kubectl label po kubia-manual-v2 env=debug --overwrite

查看namespace为docker中的pod，不加-n，默认是default(-n 和--namespace等价)
> kubectl get po -n docker

一个误区：
你可能会认为当不同的用户在不同的命名空间中部署pod时，这些pod应该彼此隔离，并且无法通信，但事实却并非如此。命名空间之间是否提供网络隔离取决于Kubernetes所使用的网络解决方案。当该解决方案不提供命名空间间的网络隔离时，如果命名空间foo中的某个pod知道命名空间 bar中pod的IP地址，那它就可以将流量（例如HTTP请求）发送到另一个pod。

#### 删除po
删除po
> kubectl delete po kubia-xxx

使用标签选择器删除pod(一次删除所有金丝雀pod)
> kubectl delete po -l rel=canary

删除的说明

- 在删除pod的过程中，实际上我们在指示Kubernetes终止该pod中的所有容器。Kubernetes向进程发送一个SIGTERM信号并等待一定的秒数（默认为30），使其正常关闭。如果它没有及时关闭，则通过SIGKILL终止该进程。因此，为了确保你的进程总是正常关闭，进程需要正确处理SIGTERM信号
- 删除整个命名空间（pod将会伴随命名空间自动删除）

#### 查错

>  kubectl describe pod xxxx

一些概念
- 如果退出代码为137，这有特殊的含义 —— 表示该进程由外部信号终止。数字137是两个数字的总和：128+x，其中x是终止进程的信号编号。在这个例子中，x等于9，这是SIGKILL的信号编号，意味着这个进程被强行终止。
- 当容器被强行终止时，会创建一个全新的容器——而不是重启原来的容器
- 很多场合都会看到这种情况，用户很困惑为什么他们的容器正在重启。但是如果使用kubectl describe，他们会看到容器以退出码137或143结束，并告诉他们该pod是被迫终止的。此外，pod事件的列表将显示容器因liveness探测失败而被终止。如果你在pod启动时看到这种情况，那是因为未能适当设置initialDelaySeconds。
- 退出代码137表示进程被外部信号终止，退出代码为128+9（SIGKILL）。同样，退出代码143对应于128+15（SIGTERM）。

### ReplicationController

