
# k8s-workshop-ru: Part II - Настройка среды

Мастер-класс по написанию микросервисов на Go с автоматическими 
релизами в Kubernetes

## Подготовка окружения

Для второй части мастер-класса нам понадобятся:
- любой терминал SSH
- git
- Github персональная учетная запись
- консольная утилита для просмотра и управления ресурсами Kubernetes (kubectl).

Если вы принимали участие в первой части мастер-класса, то у вас уже должен быть подготовлен терминал SSH, настроен git и вы уже используете Github account.

## Интеграция с Kubernetes и Github

- Интеграция с Kubernetes через Github
  - Активируем интеграцию по ссылке [https://k8s.community](https://k8s.community)
  - После активации на Github, на странице [https://k8s.community](https://k8s.community) будет отображена информация о вашей успешной регистрации в среде Kubernetes и конфигурация для настройки доступа.

## Установка консольных утилит

- В терминале скачайте утилиту kubectl и скопируйте её в место, где у вас расположены исполняемые файлы:

  - OS X
	```sh
	curl -Lo /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.5.6/bin/darwin/amd64/kubectl
	chmod +x /usr/local/bin/kubectl
	```
	или
	```sh
	brew install kubectl
	```

  - Linux
	```sh
	curl -Lo /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.5.6/bin/linux/amd64/kubectl
	chmod +x /usr/local/bin/kubectl
	```

  - Windows
	```sh
	curl -LOo %USERPROFILE% https://storage.googleapis.com/kubernetes-release/release/v1.5.6/bin/windows/amd64/kubectl.exe
	```

## Настройка консольных утилит

- Настройка прав доступа к кластеру Kubernetes

  - OSX и Linux. Создайте файл настройки в домашнем каталоге.
	```sh
	mkdir ~/.kube
	cat <<EOF > ~/.kube/config
    apiVersion: v1
	kind: Config
	clusters:
	- cluster:
	    server: https://master.k8s.community
	  name: master-class
	contexts:
	- context:
	    cluster: master-class
	    user: master-class
	  name: master-class
	current-context: master-class
	users:
	- name: master-class
	  user:
	    token: XXXXXXX
	EOF
	```

  - Windows. Создайте файл настройки в домашнем каталоге
    ```sh
    mkdir %USERPROFILE%/.kube
    touch %USERPROFILE%/.kube/config
    ```
    (используйте любой редактор для внесения изменений в файл, например notepad)
    ```sh
    apiVersion: v1
	kind: Config
	clusters:
	- cluster:
	    server: https://master.k8s.community
	  name: master-class
	contexts:
	- context:
	    cluster: master-class
	    user: master-class
	  name: master-class
	current-context: master-class
	users:
	- name: master-class
	  user:
	    token: XXXXXXX
    ```

## Проверка работоспособности настроек

- Введите команду в терминале:
  ```sh
  kubectl get nodes
  ```

  Результат должен быть таким:
  ```sh
  NAME            STATUS                     AGE
  k8s-master-01   Ready,SchedulingDisabled   5d
  k8s-node-01     Ready                      5d
  k8s-node-02     Ready                      5d
  k8s-node-03     Ready                      5d
  ```

## Регистрация сертификата для доступа к серверу (опционально)

- Создайте файл сертификата в домашнеи каталоге.
  
  - OSX и Linux
	```sh
	cat <<EOF > ~/kube-ca.crt
	-----BEGIN CERTIFICATE-----
	MIIEkjCCA3qgAwIBAgIQCgFBQgAAAVOFc2oLheynCDANBgkqhkiG9w0BAQsFADA/
	MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
	DkRTVCBSb290IENBIFgzMB4XDTE2MDMxNzE2NDA0NloXDTIxMDMxNzE2NDA0Nlow
	SjELMAkGA1UEBhMCVVMxFjAUBgNVBAoTDUxldCdzIEVuY3J5cHQxIzAhBgNVBAMT
	GkxldCdzIEVuY3J5cHQgQXV0aG9yaXR5IFgzMIIBIjANBgkqhkiG9w0BAQEFAAOC
	AQ8AMIIBCgKCAQEAnNMM8FrlLke3cl03g7NoYzDq1zUmGSXhvb418XCSL7e4S0EF
	q6meNQhY7LEqxGiHC6PjdeTm86dicbp5gWAf15Gan/PQeGdxyGkOlZHP/uaZ6WA8
	SMx+yk13EiSdRxta67nsHjcAHJyse6cF6s5K671B5TaYucv9bTyWaN8jKkKQDIZ0
	Z8h/pZq4UmEUEz9l6YKHy9v6Dlb2honzhT+Xhq+w3Brvaw2VFn3EK6BlspkENnWA
	a6xK8xuQSXgvopZPKiAlKQTGdMDQMc2PMTiVFrqoM7hD8bEfwzB/onkxEz0tNvjj
	/PIzark5McWvxI0NHWQWM6r6hCm21AvA2H3DkwIDAQABo4IBfTCCAXkwEgYDVR0T
	AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAYYwfwYIKwYBBQUHAQEEczBxMDIG
	CCsGAQUFBzABhiZodHRwOi8vaXNyZy50cnVzdGlkLm9jc3AuaWRlbnRydXN0LmNv
	bTA7BggrBgEFBQcwAoYvaHR0cDovL2FwcHMuaWRlbnRydXN0LmNvbS9yb290cy9k
	c3Ryb290Y2F4My5wN2MwHwYDVR0jBBgwFoAUxKexpHsscfrb4UuQdf/EFWCFiRAw
	VAYDVR0gBE0wSzAIBgZngQwBAgEwPwYLKwYBBAGC3xMBAQEwMDAuBggrBgEFBQcC
	ARYiaHR0cDovL2Nwcy5yb290LXgxLmxldHNlbmNyeXB0Lm9yZzA8BgNVHR8ENTAz
	MDGgL6AthitodHRwOi8vY3JsLmlkZW50cnVzdC5jb20vRFNUUk9PVENBWDNDUkwu
	Y3JsMB0GA1UdDgQWBBSoSmpjBH3duubRObemRWXv86jsoTANBgkqhkiG9w0BAQsF
	AAOCAQEA3TPXEfNjWDjdGBX7CVW+dla5cEilaUcne8IkCJLxWh9KEik3JHRRHGJo
	uM2VcGfl96S8TihRzZvoroed6ti6WqEBmtzw3Wodatg+VyOeph4EYpr/1wXKtx8/
	wApIvJSwtmVi4MFU5aMqrSDE6ea73Mj2tcMyo5jMd6jmeWUHK8so/joWUoHOUgwu
	X4Po1QYz+3dszkDqMp4fklxBwXRsW10KXzPMTZ+sOPAveyxindmjkW8lGy+QsRlG
	PfZ+G6Z6h7mjem0Y+iWlkYcV4PIWL1iwBi8saCbGS5jN2p8M+X+Q7UNKEkROb3N6
	KOqkqm57TH2H3eDJAkSnh6/DNFu0Qg==
	-----END CERTIFICATE-----
	EOF
    ```

  - Windows.
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
