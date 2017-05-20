
# k8s-workshop-ru: Part II - Steps 1 - X

Мастер-класс по написанию микросервисов на Go с автоматическими 
релизами в Kubernetes

## Step 1 - Знакомство с Kubernetes

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

- Pods создаются посредством Deployment, Replication Controller или просто статическим манифестом.
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

- Важный компонент Service, который является основным звеном для получения доступа к Pods
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
  
  Также есть еще много компонентов, знакомство с которыми у нас состоится позже

## Step 2 - Знакомство с шаблонами charts 

- Копирование подготовленного шаблона для типичного сервиса
  - Делаем fork репозитория [mycharts](https://github.com/k8s-community/mycharts) на Github
  - Для первого релиза сервиса никаких изменений в данном репозитории делать не нужно

- Регистрируем наш репозиторий на Github в Github Pages

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

- Довольно часто, для сервисов, которые имеют публичный доступ или REST интерфейс используется шаблон Ingress
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

- Также для хранения конфигураций применяются ConfigMap
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: nginx-conf
    namespace: kube-system
  data:
    body-size: "0"
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

## Step 3 - Создание собственного сервиса для выполнения в среде Kubernetes

- Копирование подготовленного типичного сервиса ( или используем сервис, который был реализован в первой части )
  - Делаем fork репозитория [myapp](https://github.com/k8s-community/mycharts) на Github

- Кроме самого сервиса, основные компоненты, влияющие на сборку:
  - Dockerfile, в нашем примере минимизирован до FROM scratch
  - Makefile, несёт на себе основную нагрузку в выполнении этапов CI/CD

- Более подробно остановимся на командах Makefile, которые не были рассмотрены в первой части
  - `make push` - выполняет сборку `docker image` и доставляет его в `docker registry`
  - `make deploy` - выполняет доставку сервиса в среду Kubernetes, а именно работу с пакетным менеджером Helm

## Step 4 - Интеграция с Kubernetes и Github

- Интеграция с Kubernetes через Github
  - Активируем интеграцию по ссылке [https://k8s.community](https://k8s.community)
  - После активации на Github, на странице [https://k8s.community](https://k8s.community) будет отображена информация о вашей успешной регистрации в среде Kubernetes и конфигурация для настройки доступа.
  - Используем ссылку снизу `k8s_integration` для выбора репозитория на котором будем активирован CI/CD 

## Step 5 - Создание релизных веток в Git (для активации CI/CD)

- Клонируем наш сервис локально (обратите внимание, прежде чем выполнить комманду вам необходимо заменить `имя пользователя` на имя вашего аккаунта Github)
  ```sh
  git clone https://github.com/`имя пользователя`/myapp.git
  ```

- Также необходимо заменить значение `USERSPACE` внутри `Makefile` вашего сервиса `myapp` на имя аккаунта Github

- Затем создаём специальный бранч, на который будет реагировать наша система CI/CD (не забудьте указать правильный номер версии в названии бранча)
  ```sh
  git checkout -b workshop/0.1.1
  ```

- В `Makefile` затем нужно заменить значение в `RELEASE` на следующую версию сервиса

- В `myapp.go` делаем изменения в импорте `user_name` согласно аккаунту Github `"github.com/user_name/myapp/version"`

- Сохраняем наши изменения
  ```sh
  git commit -a -m “Bumped version number to 0.1.1”
  ```

- Тэгируем нашу ветку номером версии
  ```sh
  git tag -a 0.1.1 -m "Version 0.1.1"
  ```

## Step 6 - Активация CI/CD

- Отправляем нашу ветку с указанной версией в удаленный репозиторий Github 
  ```sh
  git push --set-upstream origin workshop/0.1.1
  ```
- После отправки бранча на удалённый репозиторий система CI/CD автоматически его протестирует и развернёт в среде Kubernetes. Информация об успехе или ошибках сборки будет отображена на Github в разделе `Commits` ветки `workshop/0.1.1`

## Step 7 - Проверка работоспособности сервиса

- После окончания процесса деплоймента ваш сервис будет доступен по адресу `https://services.k8s.community/user_name/myapp` (`user_name` необходимо заменить на имя аккаунта Github)
