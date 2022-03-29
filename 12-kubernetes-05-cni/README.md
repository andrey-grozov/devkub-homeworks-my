## Домашнее задание к занятию "12.5 Сетевые решения CNI"

После работы с Flannel появилась необходимость обеспечить безопасность для приложения. Для этого лучше всего подойдет Calico.

### Задание 1: установить в кластер CNI плагин Calico
Для проверки других сетевых решений стоит поставить отличный от Flannel плагин — например, Calico. Требования:

установка производится через ansible/kubespray;
после применения следует настроить политику доступа к hello-world извне. Инструкции kubernetes.io, Calico

Создаем три деплоймента: [hello-world](./main/hello-world.yaml), [hello1](./main/hello1.yaml), [hello2](./main/hello2.yaml). Все они слушают 80 и 443 порт и доступны друг для друга.

    root@cp1:/home/yc-user# kubectl get po,svc -o wide
    NAME                              READY   STATUS    RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
    pod/hello-world-f96c4b6b6-4bmpj   1/1     Running   0          5m49s   10.233.90.1   node1   <none>           <none>
    pod/hello1-86b9b85db8-vwnbl       1/1     Running   0          5m49s   10.233.96.3   node2   <none>           <none>
    pod/hello2-6cc67bb9cf-b5w4b       1/1     Running   0          5m49s   10.233.90.2   node1   <none>           <none>

    NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE     SELECTOR
    service/hello-world   ClusterIP   10.233.29.255   <none>        80/TCP,443/TCP   5m49s   app=hello-world
    service/hello1        ClusterIP   10.233.52.139   <none>        80/TCP,443/TCP   5m49s   app=hello1
    service/hello2        ClusterIP   10.233.3.205    <none>        80/TCP,443/TCP   5m49s   app=hello2
    service/kubernetes    ClusterIP   10.233.0.1      <none>        443/TCP          94m     <none>

Проверим соединения по 80 и 443 портам:

    root@cp1:/home/yc-user# kubectl exec hello-world-f96c4b6b6-4bmpj -- curl -s -m 1 -k https://hello1
    Praqma Network MultiTool (with NGINX) - hello1-86b9b85db8-vwnbl - 10.233.96.3

    root@cp1:/home/yc-user# kubectl exec hello1-86b9b85db8-vwnbl -- curl -s -m 1 -k https://hello2
    Praqma Network MultiTool (with NGINX) - hello2-6cc67bb9cf-b5w4b - 10.233.90.2

    root@cp1:/home/yc-user# kubectl exec hello2-6cc67bb9cf-b5w4b -- curl -s -m 1 http://hello-world
    Praqma Network MultiTool (with NGINX) - hello-world-f96c4b6b6-4bmpj - 10.233.90.1


Применим политику [10-hello-world](./network-policy/10-hello-world.yaml), которая запрещает для hello2 доступ по 80 порту, а hello1 разрешает и 80 и 443 порт к hello-world:

    root@cp1:/home/yc-user# kubectl apply -f ./network-policy/10-hello-world.yaml
    networkpolicy.networking.k8s.io/hello-world created

    root@cp1:/home/yc-user# kubectl exec hello2-6cc67bb9cf-b5w4b -- curl -s -m 1  http://hello-world
    command terminated with exit code 28

    root@cp1:/home/yc-user# kubectl exec hello2-6cc67bb9cf-b5w4b -- curl -s -m 1 -k https://hello-world
    Praqma Network MultiTool (with NGINX) - hello-world-f96c4b6b6-4bmpj - 10.233.90.1

Применим политику [20-hello1.yaml](./network-policy/20-hello1.yaml), она разрешает входящие только от hello-world

    root@cp1:/home/yc-user# kubectl apply -f ./network-policy/20-hello1.yaml
    networkpolicy.networking.k8s.io/hello1 created

    root@cp1:/home/yc-user# kubectl exec hello2-6cc67bb9cf-b5w4b -- curl -s -m 1  http://hello1
    command terminated with exit code 28

    root@cp1:/home/yc-user# kubectl exec hello-world-f96c4b6b6-4bmpj -- curl -s -m 1 -k https://hello1
    Praqma Network MultiTool (with NGINX) - hello1-86b9b85db8-vwnbl - 10.233.96.3

Применим политику [30-hello1.yaml](./network-policy/30-hello2.yaml), которая запретит доступ от hello2 к hello1 и разрешит входящие к hello2 от hello-world

    root@cp1:/home/yc-user# kubectl apply -f ./network-policy/30-hello2.yaml
    networkpolicy.networking.k8s.io/hello2 created

    root@cp1:/home/yc-user# kubectl exec hello2-6cc67bb9cf-b5w4b -- curl -s -m 1 http://hello1
    command terminated with exit code 28

    root@cp1:/home/yc-user# kubectl exec hello-world-f96c4b6b6-4bmpj -- curl -s -m 1 -k https://hello2
    Praqma Network MultiTool (with NGINX) - hello2-6cc67bb9cf-b5w4b - 10.233.90.2

### Задание 2: изучить, что запущено по умолчанию
Самый простой способ — проверить командой calicoctl get . Для проверки стоит получить список нод, ipPool и profile. Требования:

установить утилиту calicoctl;
получить 3 вышеописанных типа в консоли.

    root@cp1:/home/yc-user# curl -L https://github.com/projectcalico/calico/releases/download/v3.22.1/calicoctl-linux-amd64 -o calicoctl
    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    100   658  100   658    0     0   2030      0 --:--:-- --:--:-- --:--:--  2030
    100 38.5M  100 38.5M    0     0  18.3M      0  0:00:02  0:00:02 --:--:-- 31.4M
    root@cp1:/home/yc-user# chmod +x ./calicoctl
    
    root@cp1:/home/yc-user# calicoctl --allow-version-mismatch get nodes
    NAME
    cp1
    node1
    node2

    root@cp1:/home/yc-user# calicoctl --allow-version-mismatch get ipPool
    NAME           CIDR             SELECTOR
    default-pool   10.233.64.0/18   all()

    root@cp1:/home/yc-user# calicoctl --allow-version-mismatch get profile
    NAME
    projectcalico-default-allow
    kns.default
    kns.kube-node-lease
    kns.kube-public
    kns.kube-system
    ksa.default.default
    ksa.kube-node-lease.default
    ksa.kube-public.default
    ksa.kube-system.attachdetach-controller
    ksa.kube-system.bootstrap-signer
    ksa.kube-system.calico-kube-controllers
    ksa.kube-system.calico-node
    ksa.kube-system.certificate-controller
    ksa.kube-system.clusterrole-aggregation-controller
    ksa.kube-system.coredns
    ksa.kube-system.cronjob-controller
    ksa.kube-system.daemon-set-controller
    ksa.kube-system.default
    ksa.kube-system.deployment-controller
    ksa.kube-system.disruption-controller
    ksa.kube-system.dns-autoscaler
    ksa.kube-system.endpoint-controller
    ksa.kube-system.endpointslice-controller
    ksa.kube-system.endpointslicemirroring-controller
    ...