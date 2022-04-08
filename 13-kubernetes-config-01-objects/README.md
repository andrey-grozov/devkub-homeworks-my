## Домашнее задание к занятию "13.1 контейнеры, поды, deployment, statefulset, services, endpoints"

Настроив кластер, подготовьте приложение к запуску в нём. Приложение стандартное: бекенд, фронтенд, база данных. Его можно найти в папке 13-kubernetes-config.

Создаем образы
https://hub.docker.com/repository/docker/mrgrav/some-test-frontend
https://hub.docker.com/repository/docker/mrgrav/some-test-backend

#### Задание 1: подготовить тестовый конфиг для запуска приложения
Для начала следует подготовить запуск приложения в stage окружении с простыми настройками. Требования:

под содержит в себе 2 контейнера — фронтенд, бекенд;
регулируется с помощью deployment фронтенд и бекенд;
база данных — через statefulset.

[namespases](./test-env/00-test-namespace.yaml)
[деплой фронт+бэк](./test-env/20-front-back-deployment.yaml)
[сервис фронт+бэк](./test-env/20-front-back-service.yaml)
[деплой бд](./test-env/10-db-statefulset.yaml)
[сервис бд](./test-env/20-front-back-service.yaml)

    yc-user@cp1:~$ sudo kubectl get pod,deployment,sts,svc,ep -n test
    NAME                              READY   STATUS    RESTARTS   AGE
    pod/front-back-78fb6c94ff-fg4mx   2/2     Running   0          102s
    pod/postgres-0                    1/1     Running   0          70s

    NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/front-back   1/1     1            1           102s

    NAME                        READY   AGE
    statefulset.apps/postgres   1/1     70s

    NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
    service/front-back   ClusterIP   10.233.24.133   <none>        80/TCP,9000/TCP   88s
    service/postgres     ClusterIP   10.233.59.177   <none>        5432/TCP          64s

    NAME                   ENDPOINTS                         AGE
    endpoints/front-back   10.233.96.1:80,10.233.96.1:9000   88s
    endpoints/postgres     10.233.90.2:5432                  64s

Проверим

    yc-user@cp1:~$ sudo kubectl port-forward -n test service/front-back 8080:80 9000:9000
    Forwarding from 127.0.0.1:8080 -> 80
    Forwarding from [::1]:8080 -> 80
    Forwarding from 127.0.0.1:9000 -> 9000
    Forwarding from [::1]:9000 -> 9000


    yc-user@cp1:~$ curl localhost:8080
    <!DOCTYPE html>
    <html lang="ru">
    <head>
        <title>Список</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link href="/build/main.css" rel="stylesheet">
    </head>
    <body>
        <main class="b-page">
            <h1 class="b-page__title">Список</h1>
            <div class="b-page__content b-items js-list"></div>
        </main>
        <script src="/build/main.js"></script>
    </body>
    </html>


#### Задание 2: подготовить конфиг для production окружения
Следующим шагом будет запуск приложения в production окружении. Требования сложнее:

каждый компонент (база, бекенд, фронтенд) запускаются в своем поде, регулируются отдельными deployment’ами;
для связи используются service (у каждого компонента свой);
в окружении фронта прописан адрес сервиса бекенда;
в окружении бекенда прописан адрес сервиса базы данных.

[namespases](./prod-env/00-prod-namespace.yaml)
[деплой фронт+бэк](./prod-env/30-front-deployment.yaml)
[сервис фронт+бэк](./prod-env/30-front-service.yaml)
[деплой бэк](./prod-env/20-back-deployment.yaml)
[сервис бэк](./prod-env/20-back-service.yaml)
[деплой бд](./prod-env/10-db-statefulset.yaml)
[сервис бд](./prod-env/10-db-service.yaml)



    yc-user@cp1:~$ sudo kubectl get pod,deployment,sts,svc,ep -n prod
    NAME                        READY   STATUS    RESTARTS   AGE
    pod/back-7b4dbfd778-zkq7j   1/1     Running   0          12s
    pod/front-d88b7f959-8mxc6   1/1     Running   0          87s
    pod/postgres-0              1/1     Running   0          2m10s

    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/back    1/1     1            1           112s
    deployment.apps/front   1/1     1            1           87s

    NAME                        READY   AGE
    statefulset.apps/postgres   1/1     2m10s

    NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    service/back       ClusterIP   10.233.52.124   <none>        9000/TCP   101s
    service/front      ClusterIP   10.233.38.52    <none>        80/TCP     80s
    service/postgres   ClusterIP   10.233.3.46     <none>        5432/TCP   2m4s

    NAME                 ENDPOINTS          AGE
    endpoints/back       10.233.96.3:9000   101s
    endpoints/front      10.233.90.4:80     80s
    endpoints/postgres   10.233.90.3:5432   2m4s
