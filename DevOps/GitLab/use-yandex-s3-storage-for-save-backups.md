
Регистрируемся в Yandex

Переходим по ссылке https://console.yandex.cloud/folders/

Справа посреди экрана нажимаем на синюю кнопку "Создать ресурс"

В появившемся select-поле находим и выбираем "Бакет"

Заполняем данные для бакета - указываем его имя, например, gitlabdev

Макс. размер указываем - 1 ГБ

Выбираем хранилище - стандартное

Нажимаем "Создать бакет"

Бакеты пользователя будут доступны по ссылке
https://console.yandex.cloud/folders/<id-вашей-папки>/storage/buckets

Устанавливаем Yandex Cloud CLI

Инструкция по установке доступна по ссылке:
https://yandex.cloud/ru/docs/cli/quickstart#install

Примерная команда:
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash

После завершения установки yandex cloud cli необходимо перезапустить командную оболочку (перезайти через ssh подключение)
или выполнить source "/root/.bashrc" для обновления загруженной в сессию переменной PATH

Инструкция по настройке доступа к yandex cloud cli доступна на странице https://yandex.cloud/ru/docs/cli/quickstart#yandex-account_1 

Чтобы начать настройку профиля CLI, необходимо  выполнить команду:
yc init

Будет выведена надпись вроде:
Please go to https://oauth.yandex.ru/authorize?response_type=token&client_id=<id вашего аккаунта> in order to obtain OAuth token.

Необходимо перейти по данной ссылке, откроется страница в браузере, где необходимо выбрать аккаунт, с которого будет происходить авторизация

Далее при выборе аккаунта будет совершен переход на почти пустую страницу, где по центру будет располагаться токен доступа

Его необходимо будет ввести в консоль терминала после надписи "Please enter OAuth token:"

После нажатия на Enter будет выведено:

You have one cloud available: '<название вашего аккаунта>' (id = <id вашего аккаунта>). It is going to be used by default.
Please choose folder to use:
[1] default (id = <id вашего аккаунта)
[2] Create a new folder
Please enter your numeric choice:

Необходимо ввести номер папки, введем 1

Далее подтверждаем все выборы (через y) или вводим 1 на все вопросы

Проверить настройки профиля можно через команду yc config list

Чтобы просмотреть все доступные в cli бакеты нужно использовать команду ниже:

yc storage bucket list


Создаем сервисный аккаунт:
yc iam service-account create --name <имя сервисного аккаунта>

Напишем например следующее yc iam service-account create --name gitlabdev-storage-manager

Выведется в ответ что-то вроде:

id: <id-сервисного аккаунта>
folder_id: <id-директории>
created_at: "время создания"
name: gitlabdev-storage-manager


Чтобы сервисный аккаунт мог работать с бакетами, добавь ему роль storage.editor:

yc resource-manager folder add-access-binding <id-директории> \
--role storage.editor \
--subject serviceAccount:<id-сервисного аккаунта>

Получаем в ответ что-то вроде

effective_deltas:
- action: ADD
  access_binding:
  role_id: storage.editor
  subject:
  id: <id-сервисного аккаунта>
  type: serviceAccount

Теперь создаем ключи (key_id и secret) для этого сервисного аккаунта:

yc iam access-key create --service-account-name <название сервисного аккаунта>

В нашем примере:

yc iam access-key create --service-account-name gitlabdev-storage-manager

В качестве ответа получаем следующее

access_key:
id: <id ключа доступа>
service_account_id: <id-сервисного аккаунта>
created_at: <время создания>
key_id: <значение key_id>
secret: <значение secret>

Сохраняем значения key_id и secret в надежном месте

Проверяем, что сервисный аккаунт появился через команду yc iam service-account list

Устанавливаем инструмент для работы с S3 - aws-cli через команды:

# Скачать установщик
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip

# Установить
sudo ./aws/install

# Проверить
aws --version

Настраиваем под Yandex:
aws configure --profile yandex

Отвечаем на вопросы:

Access Key ID  - <key_id>
Secret Access Key  - <secret>
Default region - ru-central1
Output format - json

Получить список бакетов можно через команду:

aws s3 ls \
--endpoint-url https://storage.yandexcloud.net \
--profile yandex

Чтобы не писать каждый раз --endpoint-url и --profile,  можно создать alias, для этого нужно добавить эти параметры в ~/.bashrc через команду ниже

alias aws-yandex='aws --endpoint-url https://storage.yandexcloud.net --profile yandex'

Перезапускаем оболочку:
source ~/.bashrc

Теперь получить список бакетов можно через команду:
aws-yandex s3 ls

Список файлов в бакете:

aws-yandex s3 ls s3://<имя-бакета>

В нашем случае:
aws-yandex s3 ls s3://gitlabdev

Загрузить файл в бакет:

aws-yandex s3 cp <путь-к-файлу> s3://<имя-бакета>/<путь-в-бакете>/

В нашем случае, например:
aws-yandex s3 cp /root/docker-compose-gitlab-yaml-config/docker-compose.yaml s3://gitlabdev/docker-compose-gitlab-yaml-config/docker-compose.yaml

Скачать файл из бакета можно через команду вида:

aws-yandex s3 cp s3://<имя-бакета>/<путь-в-бакете>/<файл> <путь-на-локальной-машине>

В нашем случае:
aws-yandex s3 cp s3://gitlabdev/docker-compose-gitlab-yaml-config/docker-compose.yaml ./docker-compose-from-bucket.yaml







