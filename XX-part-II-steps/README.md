
# k8s-workshop-ru: Part II - Steps 1 - X

Мастер-класс по написанию микросервисов на Go с автоматическими 
релизами в Kubernetes

## Step 1 - Знакомство с Kubernetes

- Основной компонент с исполняемыми сервисами в Kubernetes - Pod
  ```sh
  kubectl get pods
  ```
  Команда показывает все Pods в default namespace
  ```sh
  NAME                              READY     STATUS    RESTARTS   AGE
  charts-v0-364055246-7wqwd         1/1       Running   0          4h
  charts-v0-364055246-n2m73         1/1       Running   0          4h
  charts-v0-364055246-q4plb         1/1       Running   0          4h
  echoheaders-2987548746-3dn74      1/1       Running   0          4d
  echoheaders-2987548746-v39pc      1/1       Running   0          4d
  oauth-proxy-v0-1788409642-81207   1/1       Running   0          3h
  ```
  Все Pods в k8s-community namespace
  ```sh
  kubectl get -n k8s-community pods
  ```

## Авторы

Список авторов (отсортирован по фамилиям):

- [Елена Граховац](https://github.com/rumyantseva)
- [Игорь Должиков](https://github.com/takama)
- [Владислав Савельев](https://github.com/vsaveliev)
- [Станислав Щербаков](https://github.com/STASiAN)
