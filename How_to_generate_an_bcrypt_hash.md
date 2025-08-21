# wg-password

`wg-password` (wgpw) - скрипт для генерации bcrypt хеша на основе вашего пароля для использования в проекте

## Возможности

- Генерация хешей.
- Простая интеграция с `wg-easy`.

## Использование с Docker

Для генерации пароля следует использовать дующую команду :

```sh
docker run -it ghcr.io/ReanSn0w/amnezia-wg-easy wgpw <YOUR_PASSWORD>
PASSWORD_HASH='$2b$12$coPqCsPtcFO.Ab99xylBNOW4.Iu7OOA2/ZIboHN6/oyxca3MWo7fW'
```
Если указать пароль сразу, то утилита попросит его ввести :
```sh
docker run -it ghcr.io/ReanSn0w/amnezia-wg-easy wgpw
Enter your password:      // В скрытом виде необходимо ввести пароль
PASSWORD_HASH='$2b$12$coPqCsPtcFO.Ab99xylBNOW4.Iu7OOA2/ZIboHN6/oyxca3MWo7fW'
```

**Важно** : убедитесь, что вставляете пароль в **одиночных кавычках** когда используете `docker run` :

```bash
$ echo $2b$12$coPqCsPtcF <-- not correct
b2
$ echo "$2b$12$coPqCsPtcF" <-- not correct
b2
$ echo '$2b$12$coPqCsPtcF' <-- correct
$2b$12$coPqCsPtcF
```

**Важно** : Для использования в Docker Compose необходимо продублировать знак `$` по всей строке. Например:

``` yaml
- PASSWORD_HASH=$$2y$$10$$hBCoykrB95WSzuV4fafBzOHWKu9sbyVa34GJr8VV5R/pIelfEMYyG
```
