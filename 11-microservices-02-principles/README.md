## Домашнее задание к занятию "11.02 Микросервисы: принципы"

Вы работаете в крупной компанию, которая строит систему на основе микросервисной архитектуры. Вам как DevOps специалисту необходимо выдвинуть предложение по организации инфраструктуры, для разработки и эксплуатации.

### Задача 1: API Gateway
Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.

Решение должно соответствовать следующим требованиям:

Маршрутизация запросов к нужному сервису на основе конфигурации
Возможность проверки аутентификационной информации в запросах
Обеспечение терминации HTTPS
Обоснуйте свой выбор.

 Критерий | Centralized, Edge Gateway | Two-Tier Gateway | Microgateway | Per-Pod Gateways | Sidecar Gateways and Service Mesh 
 :----:| :----: | :----: |:----:|:----:|:----:
|SSL/TLS termination            | + | +| + | + | + | 
|Authentication                 | + | +| + | + | + |
|Authorization                  | + | +|   |   | + |
|Request routing                | + | +| + | + |   |
|Rate limiting                  | + |  | + | + |   |
|Request/response manipulation  | + |  |   |   |   |
|Centralized logging            |   | +|   | + | + |
|Tracing injection              |   | +|   |   | + |
|Load balancing                 |   | +| + |   | + |
| | подходит для монолита | не поддерживает распределенное управление | может управлять трафиком между службами
сложно добиться согласованности и контроля | шлюз для каждого модуля не выполняет никакой маршрутизации или балансировки нагрузки, поэтому его часто развертывают в сочетании с одним из предыдущих 3х решений | значительно усложняет управление|

Источник - https://www.nginx.com/blog/choosing-the-right-api-gateway-pattern/

Исходя из анализа таблицы, можно применить Microgateway.


### Задача 2: Брокер сообщений
Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.

Решение должно соответствовать следующим требованиям:

Поддержка кластеризации для обеспечения надежности
Хранение сообщений на диске в процессе доставки
Высокая скорость работы
Поддержка различных форматов сообщений
Разделение прав доступа к различным потокам сообщений
Протота эксплуатации
Обоснуйте свой выбор.
|Критерий | RabbitMQ|ActiveMQ|Qpid C++|SwiftMQ|Artemis|	Apollo|
|----|----|----|----|----|----|----|
Кластеризация	|+|	+|	+|	+|	+|	-
Хранение сообщений	|+|	+|	+|	+|	+|	+
Скорость работы	|+|	-|	-|	-|	-|	-|
Поддержка различных форматов |+|	-|	-|	-|	-|	-
Права доступа	|+|	+|	+|	+|	+|	+|
Простота эксплуатации	|+|	+|	-|	-|	+|	+

На основании таблицы подходящим вариантом будет использование RabbitMQ.