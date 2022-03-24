## Домашнее задание к занятию "12.2 Команды для работы с Kubernetes"

### Задание 1: Запуск пода из образа в деплойменте
Для начала следует разобраться с прямым запуском приложений из консоли. Такой подход поможет быстро развернуть инструменты отладки в кластере. Требуется запустить деплоймент на основе образа из hello world уже через deployment. Сразу стоит запустить 2 копии приложения (replicas=2).

Требования:

пример из hello world запущен в качестве deployment
количество реплик в deployment установлено в 2
наличие deployment можно проверить командой kubectl get deployment
наличие подов можно проверить командой kubectl get pods

Создаем deployment

    Файл hello-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: hello-deployment
    labels:
        app: hello
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: hello
    template:
        metadata:
        labels:
            app: hello
        spec:
        containers:
        - name: hello
            image: k8s.gcr.io/echoserver:1.4
            ports:
            - containerPort: 8080

Запускаем deployment 

    kubectl apply -f ./controllers/hello-deployment.yaml

Проверяем наличие deployment

    root@vagrant:/home/vagrant/gitwork/devkub-homeworks-my/12-kubernetes-02-commands# kubectl get deployments
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    hello-deployment   2/2     2            2           51s

Проверяем наличие подов:

    root@vagrant:/home/vagrant/gitwork/devkub-homeworks-my/12-kubernetes-02-commands# kubectl get pods
    NAME                               READY   STATUS    RESTARTS     AGE
    hello-deployment-6c495957-6xwqz    1/1     Running   0            5m7s
    hello-deployment-6c495957-s4qpn    1/1     Running   0            5m7s


### Задание 2: Просмотр логов для разработки
Разработчикам крайне важно получать обратную связь от штатно работающего приложения и, еще важнее, об ошибках в его работе. Требуется создать пользователя и выдать ему доступ на чтение конфигурации и логов подов в app-namespace.

Требования:

создан новый токен доступа для пользователя
пользователь прописан в локальный конфиг (~/.kube/config, блок users)
пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>)

Создаем пользователя vagrant

    root@vagrant:~# cat <<EOF | kubectl apply -f -
    > apiVersion: v1
    > kind: ServiceAccount
    > metadata:
    >   name: vagrant
    >   namespace: default
    > EOF
    serviceaccount/vagrant created

Создаем роль

    root@vagrant:~# cat <<EOF | kubectl apply -f -
    > kind: Role
    > apiVersion: rbac.authorization.k8s.io/v1
    > metadata:
    >   namespace: default
    >   name: log-role
    > rules:
    > - apiGroups: ["", "extensions", "apps"]
    >   resources: ["pods"]
    >   verbs: ["get", "logs", "describe"]
    > EOF
    role.rbac.authorization.k8s.io/log-role created

Добавляем роль к аккаунту

    root@vagrant:~# cat <<EOF | kubectl apply -f -
    eBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
    > kind: RoleBinding
    > apiVersion: rbac.authorization.k8s.io/v1
    > metadata:
    >   name: vagrant
    >   namespace: default
    > subjects:
    > - kind: ServiceAccount
    >   name: vagrant
    >   namespace: default
    > roleRef:
    >   apiGroup: rbac.authorization.k8s.io
    >   kind: Role
    >   name: view
    > EOF
    rolebinding.rbac.authorization.k8s.io/vagrant created

Список ролей

    root@vagrant:~# kubectl get rolebindings
    NAME         ROLE         AGE
    vagrant      Role/view    4h6m

    root@vagrant:~# kubectl get secrets
    NAME                   TYPE                                  DATA   AGE
    default-token-sjb6h    kubernetes.io/service-account-token   3      4d9h
    vagrant-token-k6wlb    kubernetes.io/service-account-token   3      4h21m

Конфиг пользователя

    root@vagrant:/home/vagrant# kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: /root/.minikube/ca.crt
        extensions:
        - extension:
            last-update: Mon, 21 Mar 2022 13:48:23 UTC
            provider: minikube.sigs.k8s.io
            version: v1.25.2
        name: cluster_info
        server: https://10.0.2.15:8443
    name: minikube
    contexts:
    - context:
        cluster: minikube
        extensions:
        - extension:
            last-update: Mon, 21 Mar 2022 13:48:23 UTC
            provider: minikube.sigs.k8s.io
            version: v1.25.2
        name: context_info
        namespace: default
        user: minikube
    name: minikube
    current-context: minikube
    kind: Config
    preferences: {}
    users:
    - name: minikube
    user:
        client-certificate: /root/.minikube/profiles/minikube/client.crt
        client-key: /root/.minikube/profiles/minikube/client.key
    - name: vagrant
    user:
        client-key-data: REDACTED
        token: REDACTED

Просмотр логов

    root@vagrant:~# kubectl logs hello-deployment-6c495957-6xwqz
    127.0.0.1 - - [21/Mar/2022:14:36:42 +0000] "GET / HTTP/1.1" 200 1055 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36"
    127.0.0.1 - - [21/Mar/2022:14:36:43 +0000] "GET /favicon.ico HTTP/1.1" 200 966 "http://localhost:8081/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36"
    127.0.0.1 - - [21/Mar/2022:14:36:46 +0000] "GET / HTTP/1.1" 200 1062 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36"
    127.0.0.1 - - [21/Mar/2022:14:36:46 +0000] "GET /favicon.ico HTTP/1.1" 200 966 "http://localhost:8081/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36"
    127.0.0.1 - - [21/Mar/2022:14:36:46 +0000] "GET / HTTP/1.1" 200 1062 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36"
    127.0.0.1 - - [21/Mar/2022:14:36:46 +0000] "GET /favicon.ico HTTP/1.1" 200 966 "http://localhost:8081/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36"

    root@vagrant:/home/vagrant# kubectl describe pod hello-deployment-6c495957-6xwqz
    Name:         hello-deployment-6c495957-6xwqz
    Namespace:    default
    Priority:     0
    Node:         vagrant/10.0.2.15
    Start Time:   Mon, 21 Mar 2022 08:56:30 +0000
    Labels:       app=hello
                pod-template-hash=6c495957
    Annotations:  <none>
    Status:       Running
    IP:           172.17.0.5
    IPs:
    IP:           172.17.0.5
    Controlled By:  ReplicaSet/hello-deployment-6c495957
    Containers:
    hello:
        Container ID:   docker://712e30f688f69aa43c338bc069923b9e7f8ba2b87129a97f201feaedb230aaee
        Image:          k8s.gcr.io/echoserver:1.4
        Image ID:       docker-pullable://k8s.gcr.io/echoserver@sha256:5d99aa1120524c801bc8c1a7077e8f5ec122ba16b6dda1a5d3826057f67b9bcb
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
        Started:      Mon, 21 Mar 2022 13:48:38 +0000
        Last State:     Terminated
        Reason:       Error
        Exit Code:    255
        Started:      Mon, 21 Mar 2022 08:56:32 +0000
        Finished:     Mon, 21 Mar 2022 13:44:43 +0000
        Ready:          True
        Restart Count:  1
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8n26q (ro)
    Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
    Volumes:
    kube-api-access-8n26q:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
    Type     Reason          Age                    From             Message
    ----     ------          ----                   ----             -------
    Warning  NodeNotReady    4h33m (x4 over 5h23m)  node-controller  Node is not ready
    Normal   SandboxChanged  34m                    kubelet          Pod sandbox changed, it will be killed and re-created.
    Normal   Pulled          34m                    kubelet          Container image "k8s.gcr.io/echoserver:1.4" already present on machine
    Normal   Created         34m                    kubelet          Created container hello
    Normal   Started         34m                    kubelet          Started container hello
### Задание 3: Изменение количества реплик
Поработав с приложением, вы получили запрос на увеличение количества реплик приложения для нагрузки. Необходимо изменить запущенный deployment, увеличив количество реплик до 5. Посмотрите статус запущенных подов после увеличения реплик.

Требования:

в deployment из задания 1 изменено количество реплик на 5
проверить что все поды перешли в статус running (kubectl get pods)

Увеличиваем количество реплик

    root@vagrant:/home/vagrant/gitwork/devkub-homeworks-my/12-kubernetes-02-commands# kubectl scale deployment/hello-deployment --replicas=5

Смотрим статус

    root@vagrant:/home/vagrant/gitwork/devkub-homeworks-my/12-kubernetes-02-commands# kubectl get deployments
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    hello-deployment   5/5     5            5           10m

    root@vagrant:/home/vagrant/gitwork/devkub-homeworks-my/12-kubernetes-02-commands# kubectl get pods
    NAME                               READY   STATUS    RESTARTS     AGE
    hello-deployment-6c495957-69vv5    1/1     Running   0            3m46s
    hello-deployment-6c495957-6xwqz    1/1     Running   0            13m
    hello-deployment-6c495957-b56sq    1/1     Running   0            3m46s
    hello-deployment-6c495957-g5r4g    1/1     Running   0            3m46s
    hello-deployment-6c495957-s4qpn    1/1     Running   0            13m