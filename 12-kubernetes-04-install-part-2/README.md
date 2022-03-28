## Домашнее задание к занятию "12.4 Развертывание кластера на собственных серверах, лекция 2"

Новые проекты пошли стабильным потоком. Каждый проект требует себе несколько кластеров: под тесты и продуктив. Делать все руками — не вариант, поэтому стоит автоматизировать подготовку новых кластеров.

### Задание 1: Подготовить инвентарь kubespray
Новые тестовые кластеры требуют типичных простых настроек. Нужно подготовить инвентарь и проверить его работу. Требования к инвентарю:

#### подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды;
создаем 1 мастер ноду и 2 рабочие ноды на яндексе, прописываем их в [hosts.yaml](./mycluster/hosts.yaml)

#### в качестве CRI — containerd;
вносим при необходимости изменения в файл [k8s-cluster.yml](./mycluster/group_vars/k8s_cluster/k8s-cluster.yml)

    ## Container runtime
    ## docker for docker, crio for cri-o and containerd for containerd.
    ## Default: containerd
    container_manager: containerd

#### запуск etcd производить на мастере.
редактируем файл [hosts.yaml](./mycluster/hosts.yaml)

        etcd:
        hosts:
            cp1:


Запускаем установку

    root@vagrant:/home/vagrant/gitwork/kube/kubespray# ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml -b -v

    PLAY RECAP *************************************************************************************************************
    cp1                        : ok=751  changed=152  unreachable=0    failed=0    skipped=1250 rescued=0    ignored=5
    localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    node1                      : ok=582  changed=116  unreachable=0    failed=0    skipped=775  rescued=0    ignored=2
    node2                      : ok=582  changed=116  unreachable=0    failed=0    skipped=775  rescued=0    ignored=2

Проверяем ноды

    root@cp1:/home/yc-user# kubectl version
    Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.5", GitCommit:"c285e781331a3785a7f436042c65c5641ce8a9e9", GitTreeState:"clean", BuildDate:"2022-03-16T15:58:47Z", GoVersion:"go1.17.8", Compiler:"gc", Platform:"linux/amd64"}
    Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.5", GitCommit:"c285e781331a3785a7f436042c65c5641ce8a9e9", GitTreeState:"clean", BuildDate:"2022-03-16T15:52:18Z", GoVersion:"go1.17.8", Compiler:"gc", Platform:"linux/amd64"}

    root@cp1:/home/yc-user# kubectl get nodes
    NAME    STATUS   ROLES                  AGE    VERSION
    cp1     Ready    control-plane,master   11m    v1.23.5
    node1   Ready    <none>                 9m5s   v1.23.5
    node2   Ready    <none>                 9m5s   v1.23.5

    root@cp1:/home/yc-user# kubectl create deploy nginx --image=nginx:latest --replicas=2
    deployment.apps/nginx created

    root@cp1:/home/yc-user# kubectl get po -o wide
    NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
    nginx-7c658794b9-hlpbt   1/1     Running   0          88s   10.233.96.1   node2   <none>           <none>
    nginx-7c658794b9-jxbp4   1/1     Running   0          88s   10.233.90.2   node1   <none>           <none>

    root@cp1:/home/yc-user# kubectl get po -o wide
    NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
    nginx-7c658794b9-hlpbt   1/1     Running   0          4m36s   10.233.96.1   node2   <none>           <none>
    nginx-7c658794b9-jxbp4   1/1     Running   0          4m36s   10.233.90.2   node1   <none>           <none>