# Zabbix worknote

## Входные данные + общий план работ

Поставновка задачи:
- написать плейбук Ansible для автоматической установки Zabbix Server + Zabbix Proxy + Zabbix Web UI + PostgreSQL DB на четыре разных сервера от состояния чистой ОС до состояния работоспособного Zabbix,
- написать docker-compose.yml для запуска того же, но в контейнерах Docker

## Zabbix Ansible playbook
В качестве ОС целевых хостов, на которые будем устанавливать Zabbix, выбран CentOS8, т.к. CentOS 8 поддерживает и Zabbix 5.2, и Zabbix 5.0(CentOS 7 поддерживает только Zabbix 5.0).
Так же, если посмотреть на проект https://github.com/zabbix/zabbix-docker, то в файле "docker-compose_v3_centos_pgsql_latest.yaml" мы можем увидеть, что используются images: zabbix/zabbix-server-pgsql:centos-5.2-latest, zabbix/zabbix-web-apache-pgsql:centos-5.2-latest, zabbix/zabbix-proxy-sqlite3:centos-5.2-latest, которые основаны на CentOS 8.3.2011, что подкрепляет наш выбор в пользу использования CentOS 8.

##### Подготовка перед запуском прейбуков по установке и настройке PostgreSQL и Zabbix.

Открываем файл [vars.yaml](ansible-zabbix/vars.yaml) и правим переменные при необходимости.

[Inventory](ansible-zabbix/inventory) file:
```yml
all:
  children:
    zabbix:
      children:
        zabbix-server:
          hosts:
            zabbix-srv:
        zabbix-db-postgresql:
          hosts:
            zabbix-pgsql:
        zabbix-web:
          hosts:
            zabbix-web:
        zabbix-proxy:
          hosts:
            zabbix-proxy:
```

Установим роли и коллекции из ansible galaxy и зависимости из [requirements.yml](ansible-zabbix/requirements.yml):
```
ansible-galaxy collection install -r requirements.yml
ansible-galaxy role install -r requirements.yml
```

Python в CentOS по умолчанию уже установлен, однако следует создать пользователя ansible, добавить свой ssh-ключ а так же выполнить обновление всех пакетов, чтобы потенциально в системе оставалось меньше уязвимостей.
Для этого напишем playbook [init.yaml](ansible-zabbix/init.yaml) 

Запускаем playbook [init.yaml](ansible-zabbix/init.yaml) с опцией "--ask-pass"
```
ansible-playbook init.yaml -i inventory --ask-pass
```

##### Запусе плейбуков по установке и настройке PostgreSQL и Zabbix.

[Playbook](ansible-zabbix/postgresql_12.yaml) по установке и настройке postgresql:
```
ansible-playbook postgresql_12.yaml -i inventory
```

[Playbook](ansible-zabbix/zabbix.yaml) по установке и настройке Zabbix Server, Zabbix Proxy, Zabbix Web UI:
```
ansible-playbook zabbix.yaml -i inventory
```

## Docker-compose

Файл [docker-compose.yaml](docker-compose/docker-compose.yaml) является самодокументируемым. Дополнительных комментариев не требуется.
