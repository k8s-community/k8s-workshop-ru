
# k8s-workshop-ru: Part II

Мастер-класс по написанию микросервисов на Go с автоматическими 
релизами в Kubernetes

## Подготовка окружения

Для второй части мастер-класса нам понадобятся:
- любой терминал SSH
- git
- Github персональная учетная запись
- консольная утилита для просмотра и управления ресурсами Kubernetes (kubectl).
- конслоьная утилита для просмотра и управления релизами и репозиториями Helm.

Если вы принимали участие в первой части мастер-класса, то у вас уже должен быть подготовлен терминал SSH, настроен git и вы уже используете Github account.

## Установка и настройка утилиты управления ресурсами Kubernetes

- В терминале скачайте утилиту kubectl и скопируйте её в место, где у вас расположены исполняемые файлы:

  - OS X
```sh
curl -LOo /usr/local/bin/ https://storage.googleapis.com/kubernetes-release/release/v.1.5.6/bin/darwin/amd64/kubectl
chmod +x /usr/local/bin//kubectl
```
  - Linux
```sh
curl -LOo /usr/local/bin/ https://storage.googleapis.com/kubernetes-release/release/v1.5.6/bin/linux/amd64/kubectl
chmod +x /usr/local/bin//kubectl
```

  - Windows
```sh
curl -LOo %USERPROFILE% https://storage.googleapis.com/kubernetes-release/release/v1.5.6/bin/windows/amd64/kubectl.exe
```

## Авторы

Список авторов (отсортирован по фамилиям):

- [Елена Граховац](https://github.com/rumyantseva)
- [Игорь Должиков](https://github.com/takama)
- [Владислав Савельев](https://github.com/vsaveliev)
- [Станислав Щербаков](https://github.com/STASiAN)
