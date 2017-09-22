
# k8s-workshop-ru: Part II

Мастер-класс по написанию микросервисов на Go с автоматическими 
релизами в Kubernetes

## Тема 1 - Типичный сервис для выполнения в среде Kubernetes

- Копирование подготовленного типичного сервиса
  - Делаем fork репозитория [myapp](https://github.com/k8s-community/myapp) на Github

- Основные компоненты в сервисе, влияющие на сборку:
  - Dockerfile, в нашем примере минимизирован до FROM scratch
  - Makefile, несёт на себе основную нагрузку в выполнении этапов CI/CD

- Знакомимся с кодом сервиса (для тех, кто не успел это сделать в I части)

## Тема 2 - Знакомство с CI/CD процессом

- TODO: Описание процесса, схема, взаимодействие компонентов

- Интеграция с Kubernetes через Github (Основная информация по интеграции на слайдах)
  - Активируем интеграцию по ссылке [https://k8s.community](https://k8s.community)
  - После активации на Github, на странице [https://k8s.community](https://k8s.community) будет отображена информация о вашей успешной авторизации в среде Kubernetes и конфигурация для настройки доступа.
  - Используем ссылку снизу `k8s_integration` для выбора репозитория на котором будем активирован CI/CD
  - На странице Github выбираем предложенную настройку k8s интеграции.
  - Предлагается выбрать аккаунт, где у вас находится myapp.
  - После выбора аккаунта выбираем репозиторий myapp.
  - Возможно вам будет предложено подтвердить интеграцию для myapp.
  - После успешной настройки в разделе `Settings` репозитория будут отображены параметры интеграции.

## Тема 3 - Знакомство с Kubernetes

- TODO: Краткое описание основных компонент, схема

- Настройка среды
  - Настраиваемся для получения доступа по [инструкции](https://github.com/k8s-community/k8s-workshop-ru/tree/master/02-part-II-setup)

- Основной компонент с исполняемыми сервисами в Kubernetes - Pod
  ```sh
  kubectl get pods -o wide
  ```

  Команда показывает все Pods в default namespace
  ```sh
  NAME                              READY     STATUS    RESTARTS   AGE       IP            NODE
  charts-v0-553978575-4kctk         1/1       Running   0          7m        10.20.2.28    k8s-node-01
  charts-v0-553978575-6pwhl         1/1       Running   0          7m        10.20.89.20   k8s-node-02
  charts-v0-553978575-zprhz         1/1       Running   0          7m        10.20.6.22    k8s-node-03
  echoheaders-2987548746-3dn74      1/1       Running   0          4d        10.20.6.6     k8s-node-03
  echoheaders-2987548746-v39pc      1/1       Running   0          4d        10.20.2.6     k8s-node-01
  oauth-proxy-v0-1788409642-81207   1/1       Running   0          3h        10.20.2.27    k8s-node-01
  ```

  Все Pods в k8s-community namespace
  ```sh
  kubectl get -n k8s-community pods
  ```

- Pods создаются посредством специальных контроллеров, таких как Deployment, DaemonSet и т.д. или просто статическим манифестом.
  ```sh
  kubectl get deploy
  ```

  Команда показывает все Deployments в default namespace
  ```sh
  NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
  charts-v0         3         3         3            3           7h
  echoheaders       2         2         2            2           4d
  oauth-proxy-v0    1         1         1            1           6h
  user-manager-v0   1         1         1            0           1m
  ```

- Важный контроллер Service, который является основным звеном для получения доступа к Pods
  ```sh
  kubectl get svc -o wide
  ```

  Команда показывает все Services в default namespace
  ```sh
  NAME              CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE       SELECTOR
  charts-v0         10.254.25.73    <none>        80/TCP,8082/TCP   7h        app=charts-default
  echoheaders       10.254.29.248   <none>        80/TCP            4d        app=echoheaders
  kubernetes        10.254.0.1      <none>        443/TCP           4d        <none>
  oauth-proxy-v0    10.254.30.135   <none>        80/TCP            6h        app=oauth-proxy-default
  user-manager-v0   10.254.240.50   <none>        80/TCP            12m       app=user-manager-default
  ```
  
  Также есть еще много контроллеров, знакомство с которыми у нас состоится позже

## Тема 4 - Знакомство с пакетным менеджером Helm и шаблонами charts 

- TODO: Знакомство с Helm (Описание, какая роль в нашем мастер-класс, примеры использования)

- Использование подготовленного шаблона для типичного сервиса
  - Делаем fork репозитория [mycharts](https://github.com/k8s-community/mycharts) на Github

- Каждый компонент Kubernetes может быть описан манифестом в формате YAML или JSON
  ```sh
  kubectl get svc/charts-v0 -o yaml
  ```

  Сокращённый вариант выдачи для service:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    labels:
      app: charts-default
    name: charts-v0
    namespace: default
  spec:
    selector:
      app: charts-default
    ports:
    - name: charts
      port: 80
      protocol: TCP
      targetPort: 8080
    type: ClusterIP
  ```

  Как выглядит шаблон для этого сервиса (полная версия в `templates/service.yaml` ):
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: {{ template "app.fullname" . }}
    labels:
      app: {{ template "fullname" . }}
  spec:
    selector:
      app: {{ template "fullname" . }}
    ports:
    - port: {{ .Values.service.externalPort }}
      targetPort: {{ .Values.service.internalPort }}
      protocol: TCP
      name: {{ .Values.service.name }}
    type: "{{ .Values.service.type }}"
  ```

  Откуда берутся значения в шаблоне (полная версия в `values.yaml`):
  ```yaml
  ...
  service:
    ## App container name
    ##
    name: charts
  
    ## Service Type
    ## For minikube, set this to NodePort, elsewhere use ClusterIP
    ##
    type: ClusterIP

    ## App service port
    ##
    externalPort: 80

    ## Pod exposed port
    ##
    internalPort: 8080
  ...
  ```

- Как правило в каждом модуле charts есть шаблон для Deployment или Replication Controller
  ```yaml
  apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    name: charts-v0
    namespace: default
    labels:
      app: charts-default
  spec:
    minReadySeconds: 10
    replicas: 3
    revisionHistoryLimit: 5
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 25%
      spec:
        imagePullSecrets:
        - name: registry-pull-secret
        containers:
        - name: charts
          image: registry.k8s.community/charts:0.1.7
          imagePullPolicy: IfNotPresent
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
              scheme: HTTP
          readinessProbe:
            httpGet:
              path: /
              port: 8080
              scheme: HTTP
          ports:
          - containerPort: 8080
          - containerPort: 8082
          resources:
            limits:
              cpu: 50m
              memory: 48Mi
            requests:
              cpu: 50m
              memory: 48Mi
  ```

- Довольно часто, для сервисов, которые имеют публичный доступ или REST интерфейс используется контроллер Ingress
  ```yaml
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: charts-v0
    namespace: default
    annotations:
      ingress.kubernetes.io/rewrite-target: /
    labels:
      app: charts-default
  spec:
    rules:
    - host: services.k8s.community
      http:
        paths:
        - path: /charts
          backend:
            serviceName: charts-v0
            servicePort: 80
    tls:
    - hosts:
      - services.k8s.community
      secretName: tls-secret
  ```

- Для хранения конфигураций применяются ConfigMap
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: nginx-conf
    namespace: kube-system
  data:
    body-size: "0"
  ```

- Для хранения секретной информации используется Secret
  ```yaml
  apiVersion: v1
  kind: Secret
  metadata:
    name: my-database
    namespace: default
  type: Opaque
  data:
    username: bXktdXNlcg==
    password: bXktcGFzc3dvcmQ=
  ```

- Если нам нужно организовать доступ к сервису или базе, находящимися вне Kubernetes, используется External Service
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: galera
    namespace: default
  spec:
    externalName: mariadb.database.host
    ports:
    - port: 3306
      protocol: TCP
      targetPort: 3306
    type: ExternalName
  ```

  Более подробную информацию по шаблонам Helm можно посмотреть [здесь](https://github.com/kubernetes/helm/blob/master/docs/index.md)

- Подготовка шаблонов Charts новой версии
  - Клонируем наши шаблоны локально (обратите внимание, прежде чем выполнить комманду вам необходимо заменить `user_name` на имя вашего аккаунта Github)
    ```sh
    git clone https://github.com/`user_name`/mycharts.git
    ```
  - Переходим в каталог `mycharts`
  - Для продолжения работы переходим в branch `develop`
    ```sh
    git checkout develop
    ```
  - В файле Chart.yaml в ссылке на источник необходимо заменить значение `k8s-community` внутри `Chart.yaml` в `sources:` на имя вашего аккаунта Github
  - Также необходимо заменить значение `k8s-community` внутри `prepare.sh` на имя вашего аккаунта Github
  - Сохраним изменения в бранче `develop` (В реальных системах обычно требуется Pull Request и Review ваших изменений)
    ```sh
    git commit -a -m "Setup personal account"
    ```
  - Cоздаём новый релиз
    ```sh
    git checkout -b release/0.0.6
    ```
  - Далее меняем версию наших шаблонов в файле `Charts.yaml`
  - Выполняем комманду для создания индекса в корневом каталоге Charts
    ```sh
    ./prepare.sh
    ```
  - Сохраняем наши изменения
    ```sh
    git commit -a -m “New version 0.0.6”
    ```
  - Релизим наши изменения в `develop` и `master` (В реальных системах обычно требуется Pull Request и Review ваших изменений)
    ```sh
    git checkout develop
    git merge --no-ff release/0.0.6
    git checkout master
    git merge --no-ff release/0.0.6
    ```
  - Тэгируем наш branch номером версии
    ```sh
    git tag -a 0.0.6 -m "Version 0.0.6"
    ```
  - Сохраняем изменения в репозитории на Github
    ```sh
    git push -u --tags origin master
    git push -u origin develop
    ```

- Регистрируем наш репозиторий для Charts в Github Pages
  - Переходим в Github в репозитории `mycharts` -> Settings -> Github Pages
  - Выбираем master branch для нашей страницы Charts и сохраняем
  - Переходим в браузере на страницу https://`user_name`.github.io/mycharts (`user_name` - имя вашего аккаунта Github)
  - Проверяем по содержимому страницы, что всё настроено правильно
  - На странице https://`user_name`.github.io/mycharts/index.yaml  виден зарегистрированный chart с сервисом `myapp`

## Тема 5 - Подготовка сервиса для релизов и CI/CD

- Клонируем наш сервис локально (обратите внимание, прежде чем выполнить комманду вам необходимо заменить `user_name` на имя вашего аккаунта Github)
  ```sh
  git clone https://github.com/`user_name`/myapp.git
  ```

- Более подробно остановимся на командах Makefile, которые не были рассмотрены в первой части
  - `make push` - выполняет сборку `docker image` и доставляет его в `docker registry`
  - `make deploy` - выполняет доставку сервиса в среду Kubernetes, а именно работу с пакетным менеджером Helm

- Переходим в каталог `myapp`

- Для продолжения работы переходим на бранч `develop`
  ```sh
  git checkout develop
  ```

- В `myapp.go` делаем изменения в импорте `user_name` согласно аккаунту Github `"github.com/user_name/myapp/version"`

- В `Makefile` необходимо заменить значение `USERSPACE` на имя аккаунта Github

- В `glide.yaml` нужно заменить `k8s-community` на имя аккаунта GitHub

- Чтобы убедиться, что все изменения сделаны правильно, запускаем тесты
  ```sh
  make test
  ```

- Сохраним изменения в бранче `develop` (В реальных системах обычно требуется Pull Request и Review ваших измнений)
  ```sh
  git commit -a -m "Setup personal account"
  ```

- Затем создаём специальный бранч, на который будет реагировать наша система CI/CD (не забудьте указать правильный номер версии в названии бранча)
  ```sh
  git checkout -b release/0.1.1
  ```

- В `Makefile` затем нужно заменить значение в `RELEASE` на следующую версию сервиса

- Сохраняем наши изменения
  ```sh
  git commit -a -m “Bumped version number to 0.1.1”
  ```

- Тэгируем наш branch номером версии
  ```sh
  git tag -a 0.1.1 -m "Version 0.1.1"
  ```

## Тема 6 - Активация CI/CD

- Отправляем наш branch с указанной версией в удаленный репозиторий Github 
  ```sh
  git push -u --tags origin release/0.1.1
  ```
- После отправки бранча на удалённый репозиторий система CI/CD автоматически его протестирует и развернёт в среде Kubernetes. Информация об успехе или ошибках сборки будет отображена на Github в разделе `Commits` бранча `release/0.1.1`

## Тема 7 - Проверка работоспособности сервиса

- После окончания процесса деплоймента ваш сервис будет доступен по адресу `https://services.k8s.community/user_name/myapp` (`user_name` необходимо заменить на имя аккаунта Github)

- Проверяем наличие нашего сервиса в Pods в 3 экземплярах
  ```sh
  kubectl get -n user_name pods
  ```

- Проверяем наличие Deployment
  ```sh
  kubectl get -n user_name deploy
  ```

- Проверяем наличие Services
  ```sh
  kubectl get -n user_name svc -o wide
  ```

- Каким образом наш сервис представлен во внешний мир?
  ```sh
  kubectl get -n user_name ing -o yaml
  ```
