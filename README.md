# spinor72_infra
spinor72 Infra repository

## ДЗ №4
#### Самостоятельное задание (слайд 35)
Подключиться через хост bastion можно, например, такого вида командой:
`ssh -i ~/.ssh/appuser -o ProxyCommand="ssh -i ~/.ssh/appuser -W %h:%p appuser@35.204.145.242" appuser@10.164.0.3`

Для подключения простой командой вида `ssh someinternalhost` нужно воспользоваться возможностями файла конфигурации ssh.
Например, создать файл `~/.ssh/config` (если он еще не создан)
и добавить в него содержимое:
```
Host bastion
  Hostname 35.204.145.242
  User appuser
  IdentityFile /home/spa/.ssh/appuser

Host someinternalhost
  Hostname 10.164.0.3
  User appuser
  IdentityFile /home/spa/.ssh/appuser
  ProxyCommand ssh bastion -W %h:%p
```

#### Параметры для проверки:

bastion_IP = 35.204.145.242

someinternalhost_IP = 10.164.0.3

## ДЗ №5 Деплой тестового приложения

 - [x] Основное ДЗ
 - [x] Дополнительные задания

### В процессе сделано:

 - Создан инстанс reddit-app
 - Установлены пакеты ruby, bundle и mongodb
 - Тестовое приложение загружено из репозитория. 
 - Установлены зависимости
 - Проверена работа тестового приложения
 - Сделаны скрипты , автоматизирующие указанные выше действия 
   - для запуска из инстанса `install_ruby.sh`, `install_mongodb.sh`, `deploy.sh`
   - для автоматического развёртывания `reddit-app-init.sh`, `startup-script.sh` 

### Как запустить проект:
 - Запустить скрипт `reddit-app-init.sh` 

### Как проверить работоспособность:
 - Проверить скриптом `reddit-app-check.sh` ,что сервис отвечает (скрипт должен определеить ip-адрес созданного инстанса и проверить ответ сервера на код 200) 
 - Перейти по ссылке http://35.204.145.242:9292 . Должен открыться сайтик, на котором можно зарегистрироваться, и написать сообщение. 

_После проверки, удалить созданный инстанс и правила файвола можно скриптом `reddit-app-cleanup.sh`_


#### Дополнительное задание. Стартовый скрипт. (слайд 20)
```bash
gcloud compute instances create reddit-app\
  --boot-disk-size=10GB \
  --image-family ubuntu-1604-lts \
  --image-project=ubuntu-os-cloud \
  --machine-type=g1-small \
  --tags puma-server \
  --restart-on-failure \
  --metadata-from-file startup-script=./startup-script.sh
```
#### Дополнительное задание. Правило файволла (слайд 21)
```bash
gcloud compute firewall-rules create default-puma-server\
   --allow=tcp:9292 \
   --description="Access puma server from external network"\
   --source-ranges=0.0.0.0/0\
   --target-tags=puma-server 
```

#### Параметры для проверки:
testapp_IP = 35.204.145.242
testapp_port = 9292

## ДЗ №6 Сборка образов VM при помощи Packer

 - [x] Основное ДЗ
 - [x] Задания со *

### В процессе сделано:
 - скрипты от предыдущего задания перенесены в папку `config-scripts`
 - В папке packer созданы конфигурационные файлы `ubuntu16.json` и `immutable.json` для создания образа ВМ с помощью packer
 - Файлы параметризованы, пример установки параметров в `variables.json.example`
 - В папке `packer/scripts` размещены скрипты для установки ruby, mongodb и деплоя тестового приложения
 - В папке `packer/files` размещен systemd юнит для сервиса тестового приложения
 - Проверена работа утилиты Packer по созданию образа ВМ
 - Проверена работа ВМ созданной на основе  полученного таким способом образа.

### Как запустить проект:
 - перейти в папку packer
 - В папке packer Создать файлик `variables.json` с нужными значениями переменных (за образец взять `variables.json.example`) 
 - Создать образ командой `packer build -var-file=variables.json immutable.json`
 - Создать ВМ скриптом  `config-scripts/create-reddit-vm.sh`

### Как проверить работоспособность:
 - Проверить скриптом `check-reddit-vm.sh` (в папке `config-scripts`) ,что сервис отвечает (скрипт должен определить ip-адрес созданного инстанса и проверить ответ сервера на код 200) 
 - Перейти по ссылке http://<ip-address>:9292 . Должен открыться сайтик, на котором можно зарегистрироваться, и написать сообщение. 

_После проверки, удалить созданный инстанс и правила файвола можно скриптом `config-scripts/delete-reddit-vm.sh`_


## ДЗ №7 Практика IaC с использованием Terraform

 - [x] Основное ДЗ
 - [x] Задания со *
 - [x] Задание с **

### В процессе сделано:
 - Установлен terrafrom
 - В папке репозитории в папке terraform созданы конфигурационные файлы для запуска на основе (подготовленного ранее с помощью packer) базового шаблона ВМ и развертывания тестового приложения. 
 - Продекларированы входные переменные в файле `variables.tf` и выходные в `outputs.tf`
 - Рабочие файлы, генерируемые terraform занесены в список .gitignore
 - В папке `terraform/files` размещен скрипт для развертывания тестового приложения и systemd юнит для соответствующего сервиса
 - Проверена работа terrafrom по созданию и изменению инфраструктуры и корректность работы тестового приложения
 - Для задания со * проверена работа terrafrom по созданию и изменению метаданных проекта на примере ssh ключей. 
   - Если использовать ресурс типа `google_compute_project_metadata_item` то все ключи заменяются terraform-ом  
   - Если использовать ресурс типа `google_compute_project_metadata` то при наличии ранее созданных ключей terrafrom не добавляет ключи
 - Для задания с ** в конфигурацию добавлен балансировщик и проверена корректность его работы. Несколько  инстансов можно создать "вручную" (методом copy-paste) , но лучше использовать параметр `count` , поскольку это предотвращает дублирование кода.

### Как запустить проект:
 - В папке terraform Создать файлик `terrafrom.tfvars` с нужными значениями переменных (за образец взять `terrafrom.tfvars.example`) 
 - Перейти в папку terraform и инициализировать среду командой `terrafrom init`
 - Просмотреть план командой `terrafrom plan` и убедиться в корректности манипуляций, предлагаемых Terraform
 - Запустить процесс создания инфраструктуры командой `terraform apply`
  
### Как проверить работоспособность:
 - С помощью команды `terraform output` узнать ip-адрес балансировщика
 - Открыть в браузере страничку с этим адресом и портом 9292 и проверить работу тестового приложения
 - Подключиться к одному из запущенных инстансов и убедиться что при остановке сервиса puma (`systemctl stop puma`) по прежнему тестовое прилодение доступно на адресе балансировщика, в тоже время на адресе инстанса сервер не отвечает.  

_После проверки, удалить созданное командой `terraform destroy`_

## ДЗ №8 Terraform: ресурсы, модули, окружения и работа в команде

 - [x] Основное ДЗ
 - [x] Задания со *
 
### В процессе сделано:
 - С помощью утилиты packer созданы базовые образы:
    - reddit-db-base для сервера базы данных mongo 
    - reddit-app-base  для сервера приложений на базе ruby
 
 - В конфигурацию terraform импортировано правило файвола для подключения по ssh
 - Добавлен ресурс для присвоения статического адреса серверу тестового приложения
 - Произведено структурирование конфигурации и разделение на модули `app`, `db` и `vpc`, соответствующие конфигурации модулей размещены в подпапках terraform/modules. Добавлены переменные для модулей.
 - Для проверки механизма переиспользования с помощью модулей, настроено два окружения *prod* и *stage* в папках `terraform/prod` и `terraform/stage` соответственно, использующие разные параметры для развертывания (на примере правила файвола)
 - С помощью модуля `SweetOps/storage-bucket/google`   из удаленного реестра созданы бакеты
 - Для задания со *  в окружении prod настроено хранение состояния в бэкэнде Google Cloud Storage и проверена работа блокировки 
 - Для второго задания со * настроен провизионер в отдельном модуле `deploy` для развертывания тестового приложения. 
     В модуль `deploy` передается ip-адрес инстанса на котром должно быть развернуто прилоджение и 
     ip-адрес сервера MongoDB для хранения данных приложения, который прописывается в переменную среды `DATABASE_URL` 
     Модуль db снабжен провизионером для правки конфигурации mongo чтобы разрешить внешние подключения к базе. 
 

### Как запустить проект:
Описание для окружения prod ( stage - аналогично)
 - В папке terraform/prod  Создать файлик `terrafrom.tfvars` с нужными значениями переменных (за образец взять `terraform/prod/terrafrom.tfvars.example`) 
 - Перейти в папку terraform/prod и инициализировать среду командой `terrafrom init`
 - Просмотреть план командой `terrafrom plan` и убедиться в корректности манипуляций, предлагаемых Terraform
 - Запустить процесс создания инфраструктуры командой `terraform apply`
  
### Как проверить работоспособность:
 - С помощью команды `terraform output` узнать ip-адрес сервера с тестовым приложением `app_external_ip`
 - Открыть в браузере страничку с этим адресом и портом 9292 и проверить работу тестового приложения

_После проверки, удалить созданное командой `terraform destroy`_

