## Инструкция по установке корпоративного мессенджера для общения и командной работы UnicChat

###### версия free-1.0.0, версия документа 1.1
### Оглавление 
<!-- TOC -->
  * [Инструкция по установке корпоратвного мессенджера для общения и командной работы UnicChat](#инструкция-по-установке-корпоративного-мессенджера-для-общения-и-командной-работы-unicchat)
    * [Оглавление](#оглавление)
    * [Архитектура установки](#архитектура-установки)
    * [Обязательные компоненты](#обязательные-компоненты)
    * [Опциональные компоненты](#опциональные-компоненты)
    * [Шаг 1. Подготовка окружения](#шаг-1-подготовка-окружения)
    * [Шаг 2. Установка сторонних зависимостей](#шаг-2-установка-сторонних-зависимостей)
    * [Шаг 3. Установка и настройка БД - mongodb](#шаг-3-установка-и-настройка-бд---mongodb)
    * [Шаг 4. Создать базу и пользователя для подключения к базе](#шаг-4-создать-базу-и-пользователя-для-подключения-к-базе)
    * [Шаг 5. Запуск сервера UnicChat](#шаг-5-запуск-сервера-unicchat)
    * [Шаг 6. Настройка nginx как Proxy](#шаг-6-настройка-nginx-как-proxy)
    * [Шаг 7. Получение сертификата Let's Encrypt](#шаг-7-получение-сертификата-lets-encrypt)
    * [Шаг 8. Открыть доступы до внутренних ресурсов](#шаг-8-открыть-доступы-до-внутренних-ресурсов)
    * [Шаг 9. Создание пользователя администратора](#шаг-9-создание-пользователя-администратора)
    * [Шаг 10. Настройка push-уведомлений](#шаг-10-настройка-push-уведомлений)
    * [Шаг 11. Настройка подключения к SMTP серверу для отправки уведомдений в почту](#шаг-11-настройка-подключения-к-smtp-серверу-для-отправки-уведомдений-в-почту)
    * [Шаг 12. Настройка подключения к LDAP серверу](#шаг-12-настройка-подключения-к-ldap-серверу)
    * [Быстрый старт. Запуск на одном сервере](#быстрый-старт-запуск-на-одном-сервере)
    * [Быстрый старт. Запуск на двух серверах](#быстрый-старт-запуск-на-двух-серверах)
    * [Настройка отказоустойчивого решения](#настройка-отказоустойчивого-решения)
    * [Частые проблемы при установке](#частые-проблемы-при-установке)
    * [Клиентские приложения](#клиентские-приложения)
<!-- TOC -->

### Архитектура установки

___
#### Установка на 1-м сервере
![](./assets/1vm-unicchat-install-scheme.jpg "Архитектура установки на 1-м сервере")

#### Установка на 2-х серверах (рекомендуется для промышленного использования)
![](./assets/2vm-unicchat-install-scheme.jpg "Архитектура установки на 2-х серверах")
### Обязательные компоненты

___

##### Push шлюз

Публичный сервис компании Unicomm. Подключение к нему необходимо для отправки push-сообщений на мобильные платформы
Apple и Google.
Расположен во внешнем периметре на серверах компании. Серверу UnicChat требуются исходящие соединения к этому сервису и
не требуются входящие соединения.

##### ВКС шлюз

Публичный сервис компании Unicomm. Подключение к нему необходимо для работы аудио и видео конференций, а также
аудио-звонков.
Расположены во внешнем периметре на серверах компании. Серверу UnicChat требуются исходящие соединения к этому сервису и
не требуются входящие соединения.

##### Приложения UnicChat

Пользовательское приложение, установленное на iOS или Android платформе.
Сервер UnicChat должен иметь возможность принимать входящие сообщения от этих приложений, а также отправлять ответы.
Основное взаимодействие осуществляется через протокол HTTPS (443/TCP).
Для работы видео- и аудиозвонков необходимы протоколы STUN и TURN: входящие соединения на порты 7881/TCP и 7882/UDP,
а также входящий и исходящий трафик UDP по портам 50000-60000 (RTP-трафик).

### Опциональные компоненты

___

##### SMTP сервер

Используется для отправки OTP-сообщений, восстановлений пароля, напоминания о пропущенных сообщениях, предоставляется
вами.
Может быть использован как публичный, так и ваш собственный сервер. На схеме предполагается, что сервер находится в
вашем сегменте DMZ.
**Интеграция с SMTP не является обязательным условием.**

##### LDAP сервер

Используется для получения списка пользователей в системе. UnicChat может обслуживать как пользователей, заведенных в
LDAP каталоге,
так и внутренних пользователей в собсвенной базе. **Интеграция с LDAP не является обязательным условием**

### Шаг 1. Подготовка окружения

#### Требования к конфигурации на 20 пользователей. Приложение и БД устанавливаются на 1-й виртуальной машине

##### Конфигурация виртуальной машины

```
CPU 4 cores 1.7ghz, с набором инструкций FMA3, SSE4.2, AVX 2.0;
RAM 8 Gb;
150 Gb HDD\SDD;
```

#### Требования к конфигурации на 20-50 пользователей. Приложение и БД устанавливаются на разные виртуальные машины

##### Конфигурация виртуальной машины для приложения

```
CPU 4 cores 1.7ghz, с набором инструкций FMA3, SSE4.2;
RAM 8 Gb;
200 Gb HDD\SDD
```

##### Конфигурация виртуальной машины для БД

```
CPU 4 cores 1.7ghz, с набором инструкций FMA3, SSE4.2, AVX 2.0;
RAM 8 Gb;
100 Gb HDD\SDD
```

### Шаг 2. Установка сторонних зависимостей

Для ОС Ubuntu 20+ предлагаем воспользоваться нашими краткими инструкциями. Для других ОС воспользуйтесь инструкциями,
размещенными в сети Интернет.

1. Установить `docker` и `docker-compose `
2. Установить `nginx`.
3. Установить `certbot` и плагин `python3-certbot-nginx`.

### Шаг 3. Установка и настройка БД - mongodb

1. [Linux] На сервере БД выполните команду `grep avx /proc/cpuinfo`. Если в ответе вы не видите AVX, то вам лучше
   выбрать версию mongodb < 5.х, например, 4.4
   если AVX на вашем сервере поддерживается, рекомендуется выбрать версию mongodb > 5.х.

2. Запустить mongodb, например, используя yml файл ниже, предварительно указав ваш пароль `root` в
   параметре `setrootpassword`
3. Для этого сохраняем файл на сервере с названием `mongodb.yml` и запускаем БД
   командой `docker-compose -f mongodb.yml up -d`
4. Результат запуска можно проверить через несколько минут командой `docker-compose -f mongodb.yml logs`
5. ВАЖНО! Если вы планируете запустить БД и сервер UnicChat на разных виртуальных серверах, то в
   параметрах `MONGODB_INITIAL_PRIMARY_HOST` и `MONGODB_ADVERTISED_HOSTNAME` вам нужно указать адрес (DNS или IP) вашего
   сервера, где запускается БД.

```yml 
version: "3"
services:
  mongodb:
    image: docker.io/bitnami/mongodb:${MONGODB_VERSION:-4.4}
    container_name: unic.chat.free.db.mongo
    restart: on-failure
    volumes:
      - mongodb_data:/bitnami/mongodb
    environment:
      MONGODB_REPLICA_SET_MODE: primary
      MONGODB_REPLICA_SET_NAME: ${MONGODB_REPLICA_SET_NAME:-rs0}
      MONGODB_REPLICA_SET_KEY: ${MONGODB_REPLICA_SET_KEY:-rs0key}
      MONGODB_PORT_NUMBER: ${MONGODB_PORT_NUMBER:-27017}
      # поменять IP адрес своего сервера в MONGODB_INITIAL_PRIMARY_HOST и MONGODB_ADVERTISED_HOSTNAME
      MONGODB_INITIAL_PRIMARY_HOST: ${MONGODB_INITIAL_PRIMARY_HOST:-mongodb}
      MONGODB_INITIAL_PRIMARY_PORT_NUMBER: ${MONGODB_INITIAL_PRIMARY_PORT_NUMBER:-27017}
      MONGODB_ADVERTISED_HOSTNAME: ${MONGODB_ADVERTISED_HOSTNAME:-mongodb}
      MONGODB_ENABLE_JOURNAL: ${MONGODB_ENABLE_JOURNAL:-true}
      # указать свой пароль 
      MONGODB_ROOT_PASSWORD: "setrootpassword"
    ports:
      - "27017:27017"
    networks:
      - unic-chat-free

networks:
  unic-chat-free:
    driver: bridge

volumes:
  mongodb_data: { driver: local }
```

### Шаг 4. Создать базу и пользователя для подключения к базе

1. После того как база успешно запустилась, подключимся к контейнеру с запущенной БД. Для этого на сервере, где запущен
   docker контейнер c базой, выполним

```shell
docker exec -it unic.chat.free.db.mongo /bin/bash
```

где `unic.chat.free.db.mongo` - имя нашего контейнера, указанного в `mongodb.yml`, в инструкции `container_name`.

2. Теперь в командной строке внутри контейнера выполним подключение с помощью `mongosh`

```shell
mongosh -u root -p password
```

где `password` - это указанный вами пароль в файле `mongosh.yml` в параметре `MONGODB_ROOT_PASSWORD`
Если авторизация выполнена успешно, вы увидите приглашение `mongosh` и текст успешной авторизации, как на примере ниже.

```
Current Mongosh Log ID:	65c5d795e6d642628b94ece4
Connecting to:		mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+1.5.0
Using MongoDB:		4.4.15
Using Mongosh:		1.5.0

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2023-12-26T14:39:46.393+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
------

rs0 [direct: primary] test>
```

3. Теперь можно выполнить скрипты создания базы и пользователя. Для этого, предварительно, укажите ваши значения в
   параметрах ниже и выполните команды создания БД и пользователя.
    * `{db_name}` - название базы;
    * `{ucusername}` - пользователь, под которым будет подключаться приложение;
    * `{ucpassword}` - пароль пользователя приложения;

```spring-mongodb-json
// сначала создадим базу данных
use db_name;

//Выполним скрипт создания пользователя с назначением ему необходимых ролей
db.createUser({
user: "ucusername",
pwd: "ucpassword",
roles: [
{role: "readWrite", db: "local"},
{role: "readWrite", db: "db_name"},
{role: "dbAdmin", db: "db_name"},
{role: "clusterMonitor", db: "admin"}
]
});
```

### Шаг 5. Запуск сервера UnicChat

1. Заполните параметры ниже в yml файле и сохраните его как `unicchat.yml`
    * `{port}` - порт, на котором будет запущен сервер UnicChat (должен быть тот же что был указан для nginx);
    * `{db_name}` - название базы данных;
    * `{ucusername}` - пользователь, под которым будет подключаться сервер UnicChat к БД;
    * `{ucpassword}` - пароль пользователя;
    * `{mongodb}` - укажите адрес вашего сервера БД. Если вы запускаете сервер UnicChat и БД на одном сервере, оставьте текущее значение без изменений.

```yml
version: "3"
services:
  unic.chat.free:
    container_name: unic.chat.appserver.free
    image: index.docker.io/unicommhub/unicchat_free:1.0.0
    restart: on-failure
    environment:
      - MONGO_URL=mongodb://ucusername:ucpassword@mongodb:27017/db_name?replicaSet=rs0
      - MONGO_OPLOG_URL=mongodb://ucusername:ucpassword@mongodb:27017/local
      - ROOT_URL=http://localhost:8080
      - PORT=8080
      - DEPLOY_METHOD=docker
    ports:
# указать свой порт на котором будет доступен сервер UnicChat
      - "port:8080"
    networks:
      - unic-chat-free

networks:
  unic-chat-free:
    driver: bridge
```

2. Запустить контейнер, например, командой `docker-compose -f unicchat.yml up -d`
3. После запуска приложения, вы можете открыть веб-интерфейс приложения по адресу `http://unicchat_server_ip:port`, где `unicchat_server_ip` - имя или IP адрес сервера, 
где был запущен UnicChat, `port` - значение параметра, которые вы указали выше.
4. Теперь можно приступить к настройке сервера и создания первого пользователя (перейдите на [Шаг 9. Создание пользователя администратора](#шаг-9-создание-пользователя-администратора)) или  
   настройте прокси сервер и получите сертификат HTTPS, для этого перейдите на [Шаг 7. Настройка nginx как Proxy](#шаг-7-настройка-nginx-как-proxy)

### Шаг 6. Настройка nginx как Proxy

Для того чтобы сервер приложения не был доступен в Интернет, рекомендуется использовать proxy-сервер, для фильтрации
запросов на сервер приложения.
Мы предлагаем использовать бесплатный сервер nginx с настройкой reverse proxy.
Ниже приведен пример конфигурации. Значения в которые необходимо указать:

- `host` - IP или DNS имя сервера на котором запущен сервер UnicChat;
- `port` - порт на котором запущен сервер UnicChat;
- `domain` - ваш домен, на котором вы будете открывать чат;

Приведенная ниже конфигурация запускает виртуальный сервер, который будет принимать запросы по адресу http://domain и
перенаправлять их на `host:port`, который вы укажете. Это должен быть адрес и порт сервера UnicChat.

```nginx configuration
upstream backend {
    server host:port;
}

server {
server_name domain www.domain;

    client_max_body_size 200M;

    error_log /var/log/nginx/domain.error.log;
    access_log /var/log/nginx/domain.access.log;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header X-Nginx-Proxy true;

        proxy_redirect off;
    }

    listen 80; 
}
```

### Шаг 7. Получение сертификата Let's Encrypt
Раздел в разработке.

### Шаг 8. Открыть доступы до внутренних ресурсов

#### Входящие соединения на стороне сервера UnicChat:

Открыть порты: 
 - 8080/TCP - по-умолчанию, сервер запускается на 8080 порту и доступен http://localhost:8080, где localhost - это IP адрес сервера UnicChat;
 - 443/TCP - порт будет нужен, если вы настроили nginx с сертификатом HTTPS;

#### Исходящие соединения на стороне сервера UnicChat:

* Открыть доступ для Push-шлюза:
  * 443/TCP, на хост **push1.unic.chat**;

* Открыть доступ для ВСК сервера:
  * 443/TCP, на хост **live.unic.chat**;
  * 7881/TCP, 7882/UDP
  * (50000 - 60000)/UDP

* Открыть доступ до  внутренних ресурсов: LDAP, SMTP, DNS при необходимости использования этого функционала

### Шаг 9. Создание пользователя администратора

* `Name` - Имя пользователя, которое будет отображаться в чате;
* `Username` - Логин пользователя, который вы будете указывать для авторизации;
* `Email` - Действующая почта, используется для восстановления
* `Organization Name` - Краткое название вашей организации латинскими буквами без пробелов и спец. символов,
  используется для регистрации push уведомлений. Может быть указан позже;
* `Organization ID` - Идентификатор вашей организации, используется для подключения к push серверу. Может быть указан
  позже. Для получения ID необходимо написать запрос с указанием значения в Organization Name на почту
  support@unicomm.pro;
* `Password` - пароль вашего пользователя;
* `Confirm your password` - подтверждение пароля;

4. После создания пользователя, авторизоваться в веб-интерфейсе с использованием ранее указанных параметров.
5. Для включения пушей, перейти в раздел Администрирование - Push. Включить использование шлюза и указать адрес
   шлюза https://push1.unic.chat
6. Перейти в раздел Администрирование - Organization, убедиться что поля заполнены в соответствии с п.2
7. Настройка завершена.

При первом входе может возникнуть информационное предупреждение
![](./assets/111.jpg "Предупреждение при первом входе")

Нажмите "ДА"

### Шаг 10. Настройка push-уведомлений 
Раздел в разработке.
### Шаг 11. Настройка подключения к SMTP серверу для отправки уведомдений в почту
Раздел в разработке.
### Шаг 12. Настройка подключения к LDAP серверу
Раздел в разработке.
### Быстрый старт. Запуск на одном сервере 
Раздел в разработке.
### Быстрый старт. Запуск на двух серверах
Раздел в разработке.
### Настройка отказоустойчивого решения
Раздел в разработке.
### Частые проблемы при установке
Раздел в разработке.

### Клиентские приложения
* [Репозитории клиентских приложений]
* Android: (https://play.google.com/store/apps/details?id=pro.unicomm.unic.chat&pcampaignid=web_share)
* iOS: (https://apps.apple.com/ru/app/unicchat/id1665533885)
* Desktop:  (https://github.com/unicommorg/unic.chat.desktop.releases/releases)
