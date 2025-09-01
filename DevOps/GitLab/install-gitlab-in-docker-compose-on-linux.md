
Скачиваем программу Termius с официального сайта https://www.termius.com/

Проходим регистрацию, заводим бесплатный аккаунт

Заходим на хостинг платформу, где можно приобрести виртуальный сервер, например на платформу https://play2go.cloud/

Проходим регистрацию, заходим на страницу, где можно заказать виртуальный сервер https://play2go.cloud/me/buy

Выбираем сервер с минимумом 4 GB DDR4

Указываем название для сервера в поле "Выберите название для сервера", оно будет соответствовать нашему будущему домену

Покупаем виртуальный сервер с Ubuntu 24

На почту с аккаунта вида noreply@play2go.cloud придет сообщения с данными для доступа к серверу

Заходим в Termius, в левом верхнем углу нажимаем на кнопку new host, справа появляется форма для ввода данных
для входа на сервер, вводим IP адрес сервера, username (по умолчанию root) и пароль из сообщения на почту
листам в формочке вниз и нажимаем на connect, далее на кнопку continue

Далее необходимо установить Docker и Docker-compose на машину

Ищем на официальном сайте алгоритм установки docker на ubuntu
На текущий момент ссылка следующая https://docs.docker.com/engine/install/ubuntu/
Листаем страницу до "Install using the apt repository" (https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

Копируем команду из блока с кодом и вставляем в терминал
Подтверждаем все действия во время установки (вводим Y по необходимости)

Выполняем следующую за фразой "Install the Docker packages." команду вида:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Проверяем версию docker через docker -v, если все прошло успешно, то высветиться версия docker

Выполняем команду sudo apt install docker-compose, устанавливаем docker-compose

Открываем порты через команду sudo ufw allow 22 (ssh), 80(http), 443(https), 2424 (любые другие используемые порты)/tcp

22 порт необходимо открыть, чтобы не заблокировать подключение к серверу через ssh во время включения фаервола

Включаем фаервол через команду sudo ufw enable

Создаем папки через команды mkdir /root/docker-compose-gitlab-yaml-config, mkdir /root/docker-compose-gitlab-volumes

Устанавливаем переменную среды в постоянное использование через команду echo 'export GITLAB_HOME=/root/docker-compose-gitlab-volumes' >> ~/.bashrc

Применяем сразу же в текущей сессии терминала source ~/.bashrc

Открываем новый терминал и проверяем работу переменной через команду echo $GITLAB_HOME

Переходим в папку через команду cd /root/docker-compose-gitlab-yaml-config

Выполняем команду nano docker-compose.yaml, открывается терминальный редактор файлов

Заходим на сайт gitlab, где описывается конфигурация gitlab для docker-compose по следующей ссылке:

https://docs.gitlab.com/install/docker/installation/#install-gitlab-by-using-docker-compose

Находим содержимое примерного вида:

services:
    gitlab:
        image: gitlab/gitlab-ce:latest
        container_name: gitlab
        restart: always
        hostname: 'gitlabdev.play2go.cloud'
        environment:
            GITLAB_OMNIBUS_CONFIG: |
                external_url 'http://gitlabdev.play2go.cloud'
                gitlab_rails['gitlab_shell_ssh_port'] = 2424
        ports:
            - '80:80'
            - '443:443'
            - '2424:22'
        volumes:
            - '$GITLAB_HOME/config:/etc/gitlab'
            - '$GITLAB_HOME/logs:/var/log/gitlab'
            - '$GITLAB_HOME/data:/var/opt/gitlab'
        shm_size: '256m'

Вместо gitlabdev.play2go.cloud в hostname вставляем тот домен, который пришел в сообщении на почту с данными для входа

Соответственно делаем также с external_url, заменяем все кроме http://

В качестве image используем docker образ gitlab

Найти актуальные образы можно по ссылке https://hub.docker.com/r/gitlab/gitlab-ce/tags/

Например, есть docker образ gitlab/gitlab-ce:nightly или gitlab/gitlab-ce:18.1.5-ce.0

Указать необходимость загрузки последнего доступного образа можно через latest флаг в gitlab/gitlab-ce:latest 

Зажимаем Ctrl, нажимаем букву O на английской раскладке
Появится надпись File Name to Write: <имя файла>
Если мы не хотим изменить имя файла, то нажимаем Enter, иначе меняем имя файла и нажимаем Enter,
Чтобы выйти из редактора зажимаем Ctrl и нажимаем на X на английской раскладке

Запустить приложение можно через выполнение команды: docker compose up -d

Ее необходимо выполнить в директории с файлом docker-compose.yaml

Приложение будет запущено в фоновом режиме

Чтобы остановить приложение нужно выполнить команду: docker compose up

Ее необходимо выполнить в директории с файлом docker-compose.yaml

Зайти в приложение можно будет по ip адресу в браузере через

http://<IP адрес из сообщения с доступами к серверу, которое было отправлено на почту>

После запуска приложения необходимо узнать сгенерированный им пароль администратора

Приложение сгенерировало тома с файлами конфигурации, необходимо перейти в папку /root/docker-compose-gitlab-volumes
или другую, которая была указана в GITLAB_HOME

Там будет папка config и в ней файл initial_root_password

открываем этот файл через nano initial_root_password

Там будет строка вида Password: <пароль администратора>

Сохраняем этот пароль, он дальше будет использовать для входа в приложение
файл можно удалить или он будет сам удален через сутки после генерации