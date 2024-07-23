# Создание Docker контейнера с Ubuntu GUI

## Введение

Руководство по созданию и настройке Docker контейнера с Ubuntu GUI и VNC-сервером для удалённого доступа.


## Шаг 1: Создание Dockerfile

1. **Откройте терминал в [Docker Desktop](https://www.docker.com/products/docker-desktop/) и перейдите в директорию вашего проекта:**

    ```sh
    cd путь_к_вашей_папке\ubuntu-xfce-vnc
    ```

2. **Создайте новый файл `Dockerfile`:**

    ```sh
    New-Item -ItemType File -Name Dockerfile
    ```

3. **Откройте `Dockerfile` в текстовом редакторе и вставьте следующий код:**

    ```dockerfile
    FROM ubuntu:20.04

    # Установка переменных окружения для автоматического ответа на запросы
    ENV DEBIAN_FRONTEND=noninteractive
    ENV TZ=Etc/UTC

    # Установка нужных пакетов
    RUN apt-get update && apt-get install -y \
        xfce4 \
        xfce4-goodies \
        x11vnc \
        xvfb \
        novnc \
        supervisor \
        wget \
        curl \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*

    # Создание пользователя
    RUN useradd -m dockerUser && echo "dockerUser:password" | chpasswd && adduser dockerUser sudo

    # Установка VNC пароля
    RUN mkdir /home/dockerUser/.vnc && echo "123456" | vncpasswd -f > /home/dockerUser/.vnc/passwd && chown -R dockerUser:dockerUser /home/dockerUser/.vnc

    # Конфигурация supervisord
    COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

    # Настройка VNC и noVNC
    RUN mkdir -p /opt/noVNC/utils/websockify && \
        wget -qO- https://github.com/novnc/noVNC/archive/v1.2.0.tar.gz | tar xz --strip 1 -C /opt/noVNC && \
        wget -qO- https://github.com/novnc/websockify/archive/v0.9.0.tar.gz | tar xz --strip 1 -C /opt/noVNC/utils/websockify

    EXPOSE 5901

    USER dockerUser
    CMD ["/usr/bin/supervisord"]
    ```

## Шаг 2: Создание файла конфигурации Supervisor

1. **Создайте новый файл `supervisord.conf`:**

    ```sh
    New-Item -ItemType File -Name supervisord.conf
    ```

2. **Откройте `supervisord.conf` в текстовом редакторе и вставьте следующий код:**

    ```ini
    [supervisord]
    nodaemon=true

    [program:xvfb]
    command=/usr/bin/Xvfb :1 -screen 0 1024x768x16
    autostart=true
    autorestart=true
    user=dockerUser

    [program:x11vnc]
    command=/usr/bin/x11vnc -forever -usepw -create -display :1
    autostart=true
    autorestart=true
    user=dockerUser

    [program:startxfce4]
    command=startxfce4
    autostart=true
    autorestart=true
    user=dockerUser

    [program:novnc]
    command=/opt/noVNC/utils/novnc_proxy --vnc localhost:5901 --listen 5901
    autostart=true
    autorestart=true
    user=dockerUser
    ```

## Шаг 3: Построение и запуск Docker контейнера

1. **В терминале Docker Desktop, находясь в директории с `Dockerfile` и `supervisord.conf`, выполните команду для сборки Docker образа:**

    ```sh
    docker build -t ubuntu-xfce-vnc .
    ```

2. **После успешной сборки образа, запустите контейнер:**

    ```sh
    docker run --shm-size=256m -it -p 5901:5901 ubuntu-xfce-vnc
    ```

## Шаг 4: Подключение к VNC

1. **Используйте следующий URL для подключения через браузер:**

    ```
    http://<ваш_docker_host>:5901/?password=123456
    ```
    Вместо `<ваш_docker_host>` используйте IP-адрес или имя хоста вашего Docker сервера.

2. **Или используйте VNC Viewer для подключения по адресу:**

    ```
    <ваш_docker_host>:5901
    ```

