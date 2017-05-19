
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

- Pods создаются посредством Deployment, Replication Controller или просто статическм манифестом.
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

- Каждый компонет Kubernetes может быть описан манифестом в формате YAML или JSON
  ```sh
  kubectl get svc/charts-v0 -o yaml
  ```

  Сокращённый вариант выдачи:
  ```sh
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

  Как выглядит шаблон для этого сервиса:
  ```sh
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

  Откуда берутся значения в шаблоне:
  ```sh
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

  Более подробную информацию по шаблонам Helm можно посмотреть [здесь](https://github.com/kubernetes/helm/blob/master/docs/index.md)

## Step 3 - Создание собственного charts для сервиса в Kubernetes

- Копирование подготовленного шаблона для типичного сервиса
  - Делаем fork репозитория [mycharts](https://github.com/k8s-community/mycharts) на Github
  - Для первого релиза сервиса никаких более изменений в данном репозитории делать не нужно

- Копирование подготовленного типичного сервиса ( или используем сервис, который был реализован в первой части )
  - Делаем fork репозитория [myapp](https://github.com/k8s-community/mycharts) на Github

## Step 4 - Регистрация в Kubernetes

- Интеграция с Github
  - Активируем Github интеграцию по ссылке [https://k8s.community](https://k8s.community)
  - После активации на Github, переходим снова по ссылке [https://k8s.community](https://k8s.community) и выбираем сервис `myapp`, который у нас был создан на `Step 3` или который был реализован в первой части мастер-класса.

## Step 5 - Создание специального бранча (Активация CI/CD)

- Клонируем наш сервис локально (обратите внимание, прежде чем выполнить комманду вам необходимо заменить `имя пользователя` на то, которое у вас используется в Github)
  ```sh
  git clone https://github.com/`имя пользователя`/myapp.git
  ```
- Также необходимо заменить значение `USERSPACE` внутри `Makefile` вашего сервиса `myapp` на имя аккаунта Github
- Затем создаём специальный бранч, на который будет реагировать наша система CICD (не забудьте указать правильную версию)
  ```sh
  git checkout -b workshop/0.0.6
  ```
- Отправляем наш бранч с указанной версией в удаленный репозиторий Github 
  ```sh
  git push --set-upstream origin workshop/0.0.6
  ```
