# Дипломная работа по профессии "Системный администратор" - `Загурский Андрей`

Содержание
==========
* [Инфраструктура](#Инфраструктура)
  * [Сайт](#Сайт)
  * [Мониторинг](#Мониторинг)
  * [Логи](#Логи)
  * [Сеть](#Сеть)
  * [Резервное копирование](#Резервное-копирование)


---------
# Инфраструктура

Разработана отказоустойчивая инфраструктура для сайта, включающая мониторинг, сбор логов и резервное копирование основных данных.

Инфраструктура размещена в [Yandex Cloud](https://console.cloud.yandex.ru/cloud/b1gcvt5l6bsrvg3nfac5).

Данные для получения доступа в [Access](https://github.com/Anders1994/Diplom/blob/main/Access.md).

Конфигурация инфраструктуры выполнена с помощью Terraform и Ansible.

Файлы конфигураций описаны в [Configuration](https://github.com/Anders1994/Diplom/blob/main/Configuration.md).

## Сайт

+ Созданы `ВМ для веб-сервера` web1 и web2, в разных зонах доступности ru-central1-a и ru-central1-b соответственно.
  На них установлен nginx и использован набор статичных файлов для сайта.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Nginx.png)

+ Создана `Target Group`, в неё включены две созданные ВМ.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Target_group.png)

+ Создана `Backend Group`, настроены backends на target group, healthcheck на корень (/) и порт 80, протокол HTTP.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Backend_group.png)

+ Создан `HTTP router` для backend group, путь указан — /.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Router.png)

+ Создан `Application load balancer` для распределения трафика на веб-сервера.Задан listener тип auto, порт 80.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Balancer.png)

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Balancer_map.png)

+ Проверяем [сайт](http://130.193.41.61/):

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Curl.png)

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Site.png)

## Мониторинг

+ Создана `ВМ для Prometheus` и на ней развернут Prometheus.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Prometheus.png)

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Curl_Prometheus.png)

  Установлены `Node Exporter` и `Nginx Log Exporter` на ВМ веб-серверов.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Node_exporter.png)

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Log_exporter.png)

+ Настроен `Prometheus` на сбор метрик.

+ Создана `ВМ для Grafana` и на ней установлена Grafana.
  Настроено взаимодействие с Prometheus.
  Настроены [дашборды](http://158.160.119.107:3000/d/rYdddlPWk/node-exporter-full?orgId=1&refresh=5s) с отображением метрик USE (Utilization, Saturation, Errors) для CPU, RAM, диски, сеть, http_response_count_total, http_response_size_bytes.
  Добавлены необходимые tresholds на соответствующие графики.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Grafana_dash.png)

## Логи

+ Cоздана `ВМ для Elasticsearch`.
  Установлен filebeat в ВМ с веб-серверами, настроен на отправку access.log, error.log nginx в Elasticsearch.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Elasticsearch.png)

+ Создана `ВМ для Kibana` и на ней установлена Kibana.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Kibana.png)

  Настроено соединение с [Elasticsearch](http://158.160.99.44:5601/).

  ![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Kibana_dash.png)

## Сеть

+ Развернута `Сеть` net.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Net.png)

Сервера web1, web2, prometheus, elasticsearch находятся в приватной подсети.
Сервера grafana, kibana, application load balancer находятся в публичной подсети.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/VM.png)

+ Настроены `Security Groups` на входящий трафик.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Bastion_IN.png)

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Security_IN.png)

+ Настроена `ВМ Bastion` с публичным адресом, для нее открыт только один порт — ssh.
  Security groups настроены на разрешение входящего ssh из этой группы.
  Эта ВМ реализовывает концепцию bastion host.
  Подключение ко всем серверам по ssh осуществляется через этот хост.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Bastion_SSH.png)

## Резервное копирование

+ Настроены `Snapshot` дисков на ежедневное создание в 22:00 по МСК.
  Время жизни snapshot ограничено 7 днями.

![image](https://github.com/Anders1994/Diplom/blob/main/ScreenShots/Snapshot.png)