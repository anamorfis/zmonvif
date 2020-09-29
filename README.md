# О сервисе
Скрипт немного дописан под текущую версию zoneminder, разница с оригиналом находится в zmonvif-events.js.patch


Сервис подписывается по onvif к камере, получает уведомление о событии движения, обращается к api zoneminder на запись и по окончанию события обращается к api zoneminder для остановки записи.


# Установить
Скрипт работает похоже только на nodejs v8.

Необходимо установить nodejs, затем `n`, чтобы установить 8-ю версию nodejs


```
yum install -y git nodejs
npm install -g n
n 8
``` 


Скрипт можно распаковать в любое место, но сделать сим-линк исполняемого файла `zmonvif-events.js` в `/usr/local/bin/zmonvif-events` (либо поправить пути в файле `zmonvif/systemd/zmonvif-events.service`)


# Настройки
1. В `zmonvif-events.js` строки 4,5 пользователь в zoneminder у которого есть доступ на изменение в Monitors и включено API:
```
const zmuser = ''; 
const zmpass = '';
```
2. Systemd юнит для запуска сервиса, который будет взаимодействовать с камерой:
```
Environment=IPCAM=        # адрес IP-камеры
Environment=CAMUSER=      # пользователь на камере
Environment=CAMPASS=      # пароль пользователя на камере
Environment=ZMURL=        # урл до zoneminder, например https://172.30.1.1/zm/
Environment=ZMMONITOR=    # номер монитора камеры в zoneminder
Environment=CAMPORT=      # порт на котором камера слушает onvif
```


# Запуск
Редактируем systemd юнит, копируем/перемещаем/делаем сим-линк в `/etc/systemd/system` и запускаем, например:


1. юнит: `/etc/systemd/system/zmonvif-7.service` где 7 это номер монитора 
2. `systemctl enable --now zmonvif-7` включает автозапуск и сразу запускает сервис


Источник: https://www.npmjs.com/package/zmonvif-events

# zmonvif-events

A JS CLI tool that attempts to bridge the gap between your ONVIF camera's motion detection and Zoneminder.

## Why?
In a typical Zoneminder installation the server will do video processing to determine which frames have motion. Unfortunately this task is quite CPU intensive. 

Fortunately some ONVIF cameras have built in motion detection features, which notify subscribers when an event occurs. 

This tool connects to an ONVIF camera and subscribes to these messages. When the motion state changes, it uses Zoneminder's API to arm the selected monitor

## Install

```bash
npm install -g zmonvif-events
```

## Usage

```bash
zmonvif-events --help
usage: zmonvif-events [-h] -z ZM_BASE_URL -i ZM_MONITOR_ID -c HOSTNAME
                         [-u USERNAME] [-p PASSWORD]


ONVIF motion detection events bridge to Zoneminder

Optional arguments:
  -h, --help            Show this help message and exit.
  -z ZM_BASE_URL, --zm-base-url ZM_BASE_URL
                        Base URL for the Zoneminder instance (with trailing
                        slash)
  -i ZM_MONITOR_ID, --zm-monitor-id ZM_MONITOR_ID
                        The ID of the monitor in Zoneminder
  -c HOSTNAME, --hostname HOSTNAME
                        hostname/IP of the ONVIF camera
  -u USERNAME, --username USERNAME
                        username for the ONVIF camera
  -p PASSWORD, --password PASSWORD
                        password for the ONVIF camera
```

**Example**

```bash
  zmonvif-events \
      --zm-base-url http://my-zoneminder-instance.com/zm/ \
      --zm-monitor-id 1 \
      --hostname 192.168.1.55 \
      --username supersecretusername \
      --password dontshareme
```
```
[monitor 1]: Started
[monitor 1]: CellMotionDetector: Motion Detected: true
Setting monitor 1 to state true
[monitor 1]: CellMotionDetector: Motion Detected: false
Setting monitor 1 to state false
[monitor 1]: CellMotionDetector: Motion Detected: true
Setting monitor 1 to state true
[monitor 1]: CellMotionDetector: Motion Detected: false
```
