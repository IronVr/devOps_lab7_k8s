# devOps_lab7_k8s

## Цель работы
Освоить основы развёртывания микросервисных приложений в кластере **Kubernetes**, реализовать:
- Развёртывание локального кластера **minikube**;
- Создание контейнерного образа для **Flask-приложения** и его загрузка в окружение **minikube**;
- Написание манифестов **Deployment** и **Service** для **Flask** и **Redis**;
- Настройка балансировщика нагрузки типа **LoadBalancer** с использованием **minikube tunnel**;
- Выполнение **Rolling Update** приложения без простоя сервиса;

## Подготовка среды и сетевая настройка
* Работа выполнялась на виртуальной машине с **Ubuntu** (VirtualBox);
* Тип сети: **NAT**;
* Для доступа к сервисам с хост-машины настроен **проброс портов** в VirtualBox:
  - 8000 → 8000 (Flask-приложение)
* На гостевой машине установлен Docker, kubectl, minicube:
  ```bash
  sudo apt update
  sudo apt install docker.io docker-compose-v2

  # Скачиваем и устанавливаем kubectl
  curl -LO "https://dl.k8s.io/release/$(curl-L-s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

  # Скачиваем и устанавливаем minikube
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
  sudo dpkg -i minikube_latest_amd64.deb

  # Добавляем себе права
  sudo usermod -aG docker $USER && newgrp docker

  # Отключаем своп
  sudo swapoff-a

  # Запускаем minikube в режими однонодового кластера k8s
   minikube start--vm-driver=docker

  # Перелогиниваемся для применения настроек окружения
  su- $USER
   ```

## Создание структуры проекта

В домашней директории создана следующая структура файлов и каталогов:

    
    devOps_lab6_docker_compose/
    ├── flask_redis/ 
    │   ├── app.py
    │   ├── Dockerfile
    │   └── requirements.txt
    └── flask_redis_k8s/  
      ├── flask-deployment.yml
      ├── flask-service.yml
      ├── redis-deployment.yml
      └── redis-service.yml
        
## Запуск

   ```bash
  # Производим сборку образа из исходников прямо внутри minikube
  minikube image build-t flask:v1 flask_redis/
  # Загружаем готовый образ redis внутрь кластера
  minikube image load redis:alpine
  # Проверяем, что образы стали доступны внутри кластера
  minikube image ls
  # ИЛИ
  # Устанавливаем переменные окружения нашего шелла, чтобы команды докера перенаправлялись в кластер
  # eval $(minikube docker-env)
  # Сначала собираем образ, потом загружаем в миникуб
  # docker build-t flask:v2 flask_redis/
  # minikube image load flask:v2
  # Если миникуб не видит образ, попробуйте пересобрать с sudo (sudo docker build)
   ```

## Проверка результатов  

  ```bash
  # применяем манифесты по отдельности файлами или весь каталог целиком
  kubectl apply-f flask_redis_k8s/
  # проверяем статус развертывания реплик
  kubectl get pods
  # проверяем сервисы
  kubectl get services
  ```
<img width="789" height="296" alt="image" src="https://github.com/user-attachments/assets/a5958528-1ea8-47d6-821c-a64bd17dbbbf" />
  
  ```bash
  # разрешаем проброс трафика vm внутрь minikube
  minikube tunnel--bind-address 10.0.2.15
  ```
<img width="548" height="89" alt="image" src="https://github.com/user-attachments/assets/bf29a9dc-d9de-440c-90f5-b0dee952c37e" />

## Rolling Update

  * Обновляем app.py;
  * Пересобираем образ с новым тэгом;
  ```bash
  minikube image build -t flask:v5 flask_redis/
  ```
  * Обновляем поды на новую версию;
  ```bash
  kubectl set image deployments/flask-app flask=flask:v5
  ```
<img width="639" height="100" alt="image" src="https://github.com/user-attachments/assets/33fc5b45-c3d8-4de2-8fb8-84a3324f565b" />




