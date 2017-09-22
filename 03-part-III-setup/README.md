
# k8s-workshop-ru: Part III - Настройка среды

Мастер-класс по написанию микросервисов на Go с автоматическими 
релизами в Kubernetes

## Подготовка окружения

Для второй части мастер-класса нам понадобятся:
- любой терминал SSH
- git
- Github персональная учетная запись
- консольная утилита для просмотра и управления ресурсами Kubernetes (kubectl)

Если вы принимали участие в первой части мастер-класса, то у вас уже должен быть подготовлен терминал SSH, настроен git и вы уже используете Github account.

## Интерактивный клиент для Kubernetes

- Любители пользоваться интеракивными программами, могут скачать [Kubernetic](https://kubernetic.com)

## Установка консольных утилит

- В терминале скачайте утилиту kubectl и скопируйте её в место, где у вас расположены исполняемые файлы:

  - OS X
    ```sh
    curl -O https://storage.googleapis.com/kubernetes-release/release/v1.7.6/bin/darwin/amd64/kubectl
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    ```
    или
    ```sh
    brew install kubectl
    ```

  - Linux
    ```sh
    curl -O https://storage.googleapis.com/kubernetes-release/release/v1.7.6/bin/linux/amd64/kubectl
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    ```

  - Windows (with bash)
    ```sh
    wget http://storage.googleapis.com/kubernetes-release/release/v1.7.6/bin/windows/amd64/kubectl.exe
    chmod +x kubectl.exe
    PATH=$PATH:$(pwd)
    ```

## Настройка консольных утилит

- Настройка прав доступа к кластеру Kubernetes

  ```sh
  kubectl config set-cluster master-class --server=https://master.k8s.community
  kubectl config set-context master-class --cluster=master-class --user=master-class
  kubectl config set-credentials master-class --token=XXXXXXX
  kubectl config use-context master-class
  ```

## Регистрация сертификата для доступа к серверу

- Создайте файл сертификата в домашнеи каталоге.
  
  - OSX и Linux

    ```sh
    cat <<EOF > ~/kube-ca.crt
    -----BEGIN CERTIFICATE-----
    MIID3DCCAsSgAwIBAgIUSbHIwPypCsoJiKztI+iux+XQEwcwDQYJKoZIhvcNAQEL
    BQAwdDELMAkGA1UEBhMCTkwxEjAQBgNVBAgTCVJvdHRlcmRhbTETMBEGA1UEBxMK
    Um90dGVyZGFtIDEWMBQGA1UEChMNazhzLmNvbW11bml0eTEMMAoGA1UECxMDazhz
    MRYwFAYDVQQDEw1rOHMuY29tbXVuaXR5MB4XDTE3MDkyMTA5NTkwMFoXDTIyMDky
    MDA5NTkwMFowdDELMAkGA1UEBhMCTkwxEjAQBgNVBAgTCVJvdHRlcmRhbTETMBEG
    A1UEBxMKUm90dGVyZGFtIDEWMBQGA1UEChMNazhzLmNvbW11bml0eTEMMAoGA1UE
    CxMDazhzMRYwFAYDVQQDEw1rOHMuY29tbXVuaXR5MIIBIjANBgkqhkiG9w0BAQEF
    AAOCAQ8AMIIBCgKCAQEAwBo2PuwfdzMBqZX5WZW8y+MXVoX9g7/wyRLewMC6fEFQ
    uKl45JoHUL5QvXaXnSbcAw4PTtg+NyiYcXoGPN6GlS9IhZbkDYZuaffRTvW22A1P
    F3c4mTZKplMkX0QksGTEzrTQDH1aD/vdgpbcc7Fp+upHqTuVnFB2Zi9cDr9YpwUi
    BiohddwDtpPHnFtA/vH4rjVESbWovG4VjYxfj9bgaEXY0L02uSuq5dYOE9CIsnyV
    Liu1NWIhFgZHa1gt0SQGgJQIq+bCDTOop2ey+foyfOj6MKegluJ2hajkZx+iUhc2
    u4yGE7+td/rJ1ayvSMxUWJpbuboEr+2YJXuGVtdebwIDAQABo2YwZDAOBgNVHQ8B
    Af8EBAMCAQYwEgYDVR0TAQH/BAgwBgEB/wIBAjAdBgNVHQ4EFgQU1nuX1N4KVBXK
    IhZ0X6MBM/a5n6kwHwYDVR0jBBgwFoAU1nuX1N4KVBXKIhZ0X6MBM/a5n6kwDQYJ
    KoZIhvcNAQELBQADggEBAGmrG4yTVdpOWFRyyr3NVXUOw1ElYC4RnLytcAS7HK2I
    zzViJ77T2h5DS4OHSgLA5WRwmRYb6Rxlfz0046WDP4AWOLCzGcDLB/SUBGVAdFlF
    A4srg1FwpAS/+WkCGNIlCX3d51oRcG+/cbRNF1mZ0To7us3bEydSL4Lnoepdqdkw
    wDEZgs+EQ1bhjlwtQ/VoBDd6+RdDMKaSRhmqCdxMhCfVPp+aPehxzEJOw+i25Ylq
    JQ3gKo03ryEjvMmzt74bVO7RQA4XPo71zRhtLluIj5PrLcj5uGTADhnwn/Ma64cT
    drm9ytbafi3H5pEP14tsJ3URHwyUYIx5mMEMNrUCOH4=
    -----END CERTIFICATE-----
    EOF
    ```

  - Windows

    ```sh
    touch %USERPROFILE%/kube-ca.crt
    ```
    (используйте любой редактор для внесения изменений в файл, например notepad)

- Зарегистрируйте сертификат в настройках

  - OSX и Linux

    ```sh
    kubectl config set-cluster master-class --certificate-authority=$HOME/kube-ca.crt
    ```

  - Windows

    ```sh
    kubectl config set-cluster master-class --certificate-authority=%USERPROFILE%/kube-ca.crt
    ```

## Проверка работоспособности настроек

- Введите команду в терминале:

  ```sh
  kubectl get nodes
  ```

  Результат должен быть таким:

  ```sh
  NAME            STATUS    AGE       VERSION
  k8s-master-01   Ready     1d        v1.7.6
  k8s-master-02   Ready     1d        v1.7.6
  k8s-node-01     Ready     1d        v1.7.6
  k8s-node-02     Ready     1d        v1.7.6
  ```
