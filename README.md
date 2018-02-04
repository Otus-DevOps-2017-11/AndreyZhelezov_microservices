#######################################################

# docker-3. Домашнее задание #16.
Все рабочие файлы текущего репозитория перенесены в каталог **./docker-monolith/**.

#######################################################

# docker-2. Домашнее задание #15.
## Исходные данные.
ОС Ubuntu Linux (xenial) 16.04.
Установлена docker-machine версии 0.13.0.
В GCP создан новый проект: docker-XXXXXXXX. К нему перенастроена конфигурация gcloud.
## Работа с docker-host.
### Создание docker-host.
На GCP с помощью gcloud создана и запущена ВМ с названием docker-host. Проверка командой _docker-machine ls_ показала что хост создался и что он запущен.
Изменено окружение docker-machine на демона docker-host. Таким образом все команды docker будут выполняться демоном на docker-host.
### вопрос со \*.
Разница в выводе двух комманд:
* docker run --rm -ti tehbilly/htop;
* docker run --rm --pid host -ti tehbilly/htop;

,состоит в том что в первом (дефолтном) варинате мы увидим процесс htop созданного контейнера и только его, а во втором случае мы увидим все процессы хоста с которого запускаем контейнер. Происходит так потому что по дефолту контейнер запускается со своим **PID Namespace** и единственный процесс в нем получает PID=1, а во втором случае из-за опции _--pid=host_ процессу из контейнера выделяется PID из общего namespace хоста. И вцелом из контейнера мы получаем доступ к **PID namespace** хоста и видим все его процессы.
### Создание образа с приложением.
В репозитории создан файл конфигурации MongoDB: mongod.conf.
Создан bash-скрипт запуска приложения: start.sh.
Создан файл переменных в котором задается переменная IP-адреса хоста с БД: db_conf. 
Создан файл Dockerfile, в который будет размещено опсание нашего образа. В Dockerfile добавлены следующие комманды:
* задание исходного образа
```
FROM ubuntu:16.04
```
* обновление кэша репозиториев и установка пакетов для работы приложения
```
RUN apt-get update
RUN apt-get install -y mongodb-server ruby-full ruby-dev build-essential git
RUN gem install bundler
```
* закачка приложения в контейнер
```
RUN git clone https://github.com/Artemmkin/reddit.git
```
* копирование файлов конфигурации в контейнер
```
COPY mongod.conf /etc/mongod.conf
COPY db_config /reddit/db_config
COPY start.sh /start.sh
```
* установка зависимостей и насройка прав доступа к скрипту запуска приложения
```
RUN cd /reddit && bundle install
RUN chmod 0777 /start.sh
```
* выполнение старта приложения при старте контейнера
```
CMD ["/start.sh"]
```
На docker-host cоздан образ контейнера **reddit:latest** из описания в Dockerfile. На основе образа **reddit:latest** создан и запущен контейнер. 
При попытке подключиться к приложению, запущеному в контейнере, по внешнему адресу хоста на порт 9292 соединение не устанавливалось из-за отсутствия соответствующего правила фаервола в VPC-разделе GCP. После добавления правила доступ к приложению по http/9292 появился.
## Работа с Docker Hub.
Создана учетная запись на Docker Hub. С использованием этой учетной записи осуществлен логин из контекста docker к Docker Hub.
Далее созданный ранее образ **reddit:latest** был залит на Docker Hub для дальнейшего использования. Для этого локальному образу образу был присвоен тэг удаленного образа **<login_name>/otus-reddit:1.0** и сделан push к удаленному образу.

#######################################################

# docker-1. Домашнее задание #14.
## Исходные данные
ОС Ubuntu Linux (xenial) 16.04.
Установлена community версия Docker 17.12. Тестовый запуск показал что клиентская и серверная часть работают правильно.
## Операции с контейнерами.
После запуска тестового контейнера "Hello world" испробованы команды Docker для вывода списка контейнеров, образов. Затем на основе образа Ubuntu:16.04 было создано два контейнера и опробованы различные опции команды *docker run*.
Далее были опробованы команды для подключения к уже запущеному контейнеру и комбинации выхода из контейнера без его закрытия.
После чего был создан образ на основе контейнера, модифицированного созданием файла /tmp/file. Вывод команды docker images в котором виден созданный образ сохранен в файл docker-1.log.
### задание со *
Сравнение вывода двух команд docker inspect <image_ID> и docker inspect <container_ID> дописано в файл docker-1.log.

Далее испробованы команды завершения работы контейнеров, отображения занимаемого дискового пространства, удаления контейнеров и образов с опциями.

# AndreyZhelezov_microservices
