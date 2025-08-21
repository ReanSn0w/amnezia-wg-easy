# AmnewziaWG Easy

Вы нашли простейший способ установки и управления Amnezia WG на любом Linux хосте!

<p align="center">
  <img src="./assets/screenshot.png" width="802" />
</p>

## Возможности

* Все в одном: AmneziaWG + Web UI.
* Простая установка и использование.
* Просмотр, создание, редактирование, удаление и отключение клиентов.
* Просмотр QR кода для клиента.
* Скачивание файла конфигурации.
* Просмотр статистики подключенных клиентов.
* Tx/Rx графики для каждого клиента.
* Поддержка Gravatar или рандомных изображений.
* Автоматическое переключение режимов (темная / светлая тема)
* Поддержка разных языков
* Статистика трафика (по умолчанию: выключена)
* Одноразовые ссылки (по умолчанию: выключены)
* Ограничение по времени (по умолчанию: выключено)
* Поддержка выгрузки метрик в Prometheus

## Требования

* Хост с установленным Docker
* [Опционально] с Docker Compose

## Установка

### 1. Install Docker

- [Linux](https://docs.docker.com/engine/install/)
- [Mac & Windows](https://docs.docker.com/get-started/get-docker/)

### 2. Запуск AmneziaWG Easy

#### 2.1 С помощью Docker (менее безопасно из-за отсутсвия https)

Для автоматической установки и запуска выполни команду, подставив свои параметры:

```
  docker run -d \
  --name=amnezia-wg-easy \
  -e LANG=en \
  -e WG_HOST=<🚨YOUR_SERVER_IP> \
  -e PASSWORD_HASH=<🚨YOUR_ADMIN_PASSWORD_HASH> \
  -e PORT=51821 \
  -e WG_PORT=51820 \
  -v ~/.amnezia-wg-easy:/etc/amnezia/amneziawg \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --device=/dev/net/tun:/dev/net/tun \
  --restart unless-stopped \
  ghcr.io/ReanSn0w/amnezia-wg-easy
```

#### 2.2 Запуск с помощью Docker-comopose (С автоматической генерацией сертификата и использованием https)

> Данный способ требует использования доменного имени для выпуска сертификата

Для использования Docker Compose потребуется создать файл docker-compose.yml с содержимым

```yaml
services:
  proxy:
		image: umputun/reproxy:latest
    restart: unless-stopped
    environment:
      DOCKER_ENABLED: true
      SSL_TYPE: auto
      SSL_ACME_LOCATION: /ssl
    ports:
    	- 80:8080
    	- 443:8443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - cert_data:/ssl
    depends_on:
      awg:
        condition: service_healthy
    logging:
      driver: "json-file"
      options:
        max-size: "1M"
        max-file: "3"

  awg:
    image: ghcr.io/ReanSn0w/amnezia-wg-easy
    restart: unless-stopped
    ports:
      - 51820:51820/udp
    environment:
      WG_HOST: <🚨YOUR_SERVER_DOMAIN>
      PASSWORD_HASH: <🚨YOUR_ADMIN_PASSWORD_HASH>
      PORT: 51821
      WG_PORT: 51820
    volumes:
      - awg_data:/etc/amnezia/amneziawg
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    devices:
      - "/dev/net/tun:/dev/net/tun"
    labels:
      reproxy.server: <🚨YOUR_SERVER_DOMAIN>
      reproxy.route: ^/(.*)
      reproxy.port: 51821
    logging:
      driver: "json-file"
      options:
        max-size: "1M"
        max-file: "3"
      
volumes:
  awg_data:
  cert_data:
```

после создания файла необходимо запустить полученную конфигурацию

```bash
docker compose up -d
```



> 💡 Замени `YOUR_SERVER_IP` IP адресом своего сервера или доменом связанным с ним.
>
> 💡 Замени `YOUR_ADMIN_PASSWORD_HASH` на bcrypt хеш от твоего пароля.
> Сомтри [How_to_generate_an_bcrypt_hash.md](./How_to_generate_an_bcrypt_hash.md) что бы узнать как его получить.

UI панель будет доступна по адресу `http://<🚨YOUR_SERVER_IP>:51821`.

Prometheus доступны по адресу `http://0.0.0.0:51821/metrics`. Grafana для просмотра обычно будет на порту [21733](https://grafana.com/grafana/dashboards/21733-wireguard/)

> 💡 Твоя конфигурация хранится тут `~/.amnezia-wg-easy`

## Параметры

Полный список доступных параметров

| Env                           | Default           | Example                        | Description                                                  |
| ----------------------------- | ----------------- | ------------------------------ | ------------------------------------------------------------ |
| `PORT`                        | `51821`           | `6789`                         | Прорт для WebUI.                                             |
| `WEBUI_HOST`                  | `0.0.0.0`         | `localhost`                    | IP адрес хоста.                                              |
| `PASSWORD_HASH`               | -                 | `$2y$05$Ci...`                 | Когда параметр установлен Web UI будет запрашивать пароль для доступа. См. [How to generate an bcrypt hash.md]("https://github.com/wg-easy/wg-easy/blob/master/How_to_generate_an_bcrypt_hash.md") что бы узнать как сгенерировать хеш. |
| `WG_HOST`                     | -                 | `vpn.myserver.com`             | Публичный IP или домен связанный с хостом.                   |
| `WG_DEVICE`                   | `eth0`            | `ens6f0`                       | Ethernet устройство через которое будет проходить траффик Wireguard. |
| `WG_PORT`                     | `51820`           | `12345`                        | Публичный UDP порт сервера который будет слушать WireGuard.  |
| `WG_CONFIG_PORT`              | `51820`           | `12345`                        | UDP порт используемый для [Home Assistant Plugin](https://github.com/adriy-be/homeassistant-addons-jdeath/tree/main/wgeasy) |
| `WG_MTU`                      | `null`            | `1420`                         | MTU для клинтов. Сервер будет использовать стандартный WG MTU. |
| `WG_PERSISTENT_KEEPALIVE`     | `0`               | `25`                           | Значение в секундах во время которого соединение будет поддерживатся открытым. If this value is 0, then connections won't be kept alive. |
| `WG_DEFAULT_ADDRESS`          | `10.8.0.x`        | `10.6.0.x`                     | Сеть в рамках которой клиентам будут выдаватся адреса.       |
| `WG_DEFAULT_DNS`              | `1.1.1.1`         | `8.8.8.8, 8.8.4.4`             | DNS который будет предоставлен клиентам в конфигурации Wireguard. |
| `WG_ALLOWED_IPS`              | `0.0.0.0/0, ::/0` | `192.168.15.0/24, 10.0.1.0/24` | Список IP доступ к которым будет организован через данное соединение. |
| `WG_PRE_UP`                   | `...`             | -                              | Смотри [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L19) для просмотра значения по-умолчанию. |
| `WG_POST_UP`                  | `...`             | `iptables ...`                 | Смотри [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L19) для просмотра значения по-умолчанию. |
| `WG_PRE_DOWN`                 | `...`             | -                              | Смотри [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L19) для просмотра значения по-умолчанию. |
| `WG_POST_DOWN`                | `...`             | `iptables ...`                 | Смотри [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L19) для просмотра значения по-умолчанию. |
| `WG_ENABLE_EXPIRES_TIME`      | `false`           | `true`                         | Флаг включает возможность создавать временные конфигурации   |
| `LANG`                        | `en`              | `de`                           | Язык Web UI (Supports: en, ua, ru, tr, no, pl, fr, de, ca, es, ko, vi, nl, is, pt, chs, cht, it, th, hi). |
| `UI_TRAFFIC_STATS`            | `false`           | `true`                         | Флаг включает RX / TX статистику в Web UI                    |
| `UI_CHART_TYPE`               | `0`               | `1`                            | UI_CHART_TYPE=0 # Charts disabled, UI_CHART_TYPE=1 # Line chart, UI_CHART_TYPE=2 # Area chart, UI_CHART_TYPE=3 # Bar chart |
| `DICEBEAR_TYPE`               | `false`           | `bottts`                       | Смотри [dicebear types](https://www.dicebear.com/styles/)    |
| `USE_GRAVATAR`                | `false`           | `true`                         | Флаг активирует использование GRAVATAR                       |
| `WG_ENABLE_ONE_TIME_LINKS`    | `false`           | `true`                         | Флаг включает возможность работы с временными ссылками для скачивания документа |
| `MAX_AGE`                     | `0`               | `1440`                         | Максимальное время жизни сессии для WebUI                    |
| `UI_ENABLE_SORT_CLIENTS`      | `false`           | `true`                         | Флаг включает сортировку клиентов на основе имени            |
| `ENABLE_PROMETHEUS_METRICS`   | `false`           | `true`                         | Флаг активирует Prometheus статистику по  `http://0.0.0.0:51821/metrics` и `http://0.0.0.0:51821/metrics/json` |
| `PROMETHEUS_METRICS_PASSWORD` | -                 | `$2y$05$Ci...`                 | Когда параметр установлен Basic Auth для Prometheus будет требоватся для доступа. См. [How to generate an bcrypt hash.md]("https://github.com/wg-easy/wg-easy/blob/master/How_to_generate_an_bcrypt_hash.md") что бы узнать как сгенерировать хеш. |
| `JC`                          | `random`          | `5`                            | Кол-во пакетов с шумом, котрое будет отправлено перед сессией. |
| `JMIN`                        | `50`              | `25`                           | Минимальный размер такого пакета с шумом.                    |
| `JMAX`                        | `1000`            | `250`                          | Максимальный размер пакета с шумом.                          |
| `S1`                          | `random`          | `75`                           | Размер данных который будет добавлен к пакету инициализации соединения |
| `S2`                          | `random`          | `75`                           | Размер данных который будет добавлен к пакету ответа.        |
| `H1`                          | `random`          | `1234567891`                   | Init packet magic header — the header of the first byte of the handshake. Должен быть < uint_max. |
| `H2`                          | `random`          | `1234567892`                   | Response packet magic header — header of the first byte of the handshake response. Доблжен быть < uint_max. |
| `H3`                          | `random`          | `1234567893`                   | Underload packet magic header — UnderLoad packet header. Должен быть < uint_max. |
| `H4`                          | `random`          | `1234567894`                   | Transport packet magic header — header of the packet of the data packet. Должен быть < uint_max. |

> Если будет изменен `WG_PORT`, убедитесь что порт хоста будет соответствовать этому значению.

## Обновление

Для обновления конфигурации использующей только docker (способ 2.1):

```bash
docker stop amnezia-wg-easy
docker rm amnezia-wg-easy
docker pull ghcr.io/w0rng/amnezia-wg-easy
```

Для случая с Docker Compose

```bash
docker compose down
docker compose pull
```

А после выполнения запустить сервис подобно тому как вы делали это в пункте 2.

## Благодарности

Основано на [wg-easy](https://github.com/wg-easy/wg-easy) от Emile Nijssen.  

Использованы интеграции AmneziaWg из [amnezia-wg-easy](https://github.com/spcfox/amnezia-wg-easy) от Viktor Yudov.

Является обновленным проектом [amnezia-wg-easy](https://github.com/w0rng/amnezia-wg-easy) от [Anton Abramov](https://github.com/w0rng)
