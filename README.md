<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ | Юрчук Дмитро Володимирович |
| Група | ЗКІ-21 |
| Ідентифікатор (`sNN`) | `se-201-01` |
| Дата виконання | 25.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu |

## Параметри варіанта

<!-- Переписати з картки stand/cards/sNN.md. -->

| Параметр | Значення |
|---|---|
| `$TTL`, с | 60 |
| TTL запису SOA, с | 20 |
| Поле `minimum` SOA, с | 90 |
| Псевдонім (CNAME на `www`) | `blog` |
| Другий хост і його TTL, с | `edge`, 50 |
| Пріоритет MX | 25 |
| Токен TXT | `lab-token=3bc4ea6d` |
| Ім'я для завдання 5 | `new-wsrop.se-201-01.lab` |

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | `http` | — | `www.se-201-01.lab` | 80 (за замовч.) | `/reports/items` | `id=10&q=uk` | `section-2` |
| 2 | `http` | — | `blog.se-201-01.lab` | 80 (вказано явно) | `/items/` | `id=%D0%BA%D0%B8%D1%97%D0%B2` | — |
| 3 | `http` | `user:pass` | `edge.se-201-01.lab` | 80 (за замовч.) | `/reports` | — | `results` |
| 4 | `https` | — | `www.se-201-01.lab` | 4443 | `/reports/../items` | — | — |
| 5 | `mailto` | — | — | — | `admin@se-201-01.lab` | `subject=reports` | — |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | `www.se-201-01.lab` | `www.se-201-01.lab` | `/reports/items?id=10&q=uk` |
| 2 | `blog.se-201-01.lab` (далі за CNAME — `www.se-201-01.lab`) | `blog.se-201-01.lab` | `/items/?id=%D0%BA%D0%B8%D1%97%D0%B2` |
| 3 | `edge.se-201-01.lab` | `edge.se-201-01.lab` | `/reports` |
| 4 | `www.se-201-01.lab` | `www.se-201-01.lab:4443` | `/items` (сегмент `..` прибирає клієнт) |
| 5 | — (схема не HTTP, запит HTTP не виконується) | — | — |

Фрагмент (`#section-2`, `#results`) на сервер не надсилається, тому в цілі запиту його немає. `userinfo` з URI 3 не потрапляє ні в DNS, ні в `Host`.

Декодоване значення параметра URI 2: `київ` (байти UTF-8: `D0 BA` = к, `D0 B8` = и, `D1 97` = ї, `D0 B2` = в).

Чому URI 5 не має складової host, хоча містить символ `@`: складова authority (`userinfo@host:port`) за RFC 3986 є лише після `//`. У `mailto:` її немає, тому `admin@se-201-01.lab` цілком є складовою path, а `@` — частиною поштової адреси.

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->

```text
root@ns-se-201-01:/# curl -v 'http://www.se-201-01.lab/reports/items?id=10&q=uk#section-2'
* Host www.se-201-01.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to www.se-201-01.lab (10.53.0.81) port 80
> GET /reports/items?id=10&q=uk HTTP/1.1
> Host: www.se-201-01.lab
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Wed, 25 Nov 2026 00:12:34 GMT
< Content-Type: text/plain
< Content-Length: 134
< Connection: keep-alive
<
server: web-b
Host: www.se-201-01.lab
request URI: /reports/items?id=10&q=uk
path: /reports/items
query: id=10&q=uk
client: 10.53.1.1
* Connection #0 to host www.se-201-01.lab left intact
root@ns-se-201-01:/# curl -v 'http://blog.se-201-01.lab:80/items/?id=%D0%BA%D0%B8%D1%97%D0%B2'
* Host blog.se-201-01.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to blog.se-201-01.lab (10.53.0.81) port 80
> GET /items/?id=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: blog.se-201-01.lab
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Wed, 25 Nov 2026 00:12:58 GMT
< Content-Type: text/plain
< Content-Length: 155
< Connection: keep-alive
<
server: web-b
Host: blog.se-201-01.lab
request URI: /items/?id=%D0%BA%D0%B8%D1%97%D0%B2
path: /items/
query: id=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host blog.se-201-01.lab left intact
root@ns-se-201-01:/#
root@ns-se-201-01:/# curl -v 'http://user:pass@edge.se-201-01.lab/reports#results'
* Host edge.se-201-01.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to edge.se-201-01.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /reports HTTP/1.1
> Host: edge.se-201-01.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Wed, 25 Nov 2026 00:13:03 GMT
< Content-Type: text/plain
< Content-Length: 102
< Connection: keep-alive
<
server: web-b
Host: edge.se-201-01.lab
request URI: /reports
path: /reports
query:
client: 10.53.1.1
* Connection #0 to host edge.se-201-01.lab left intact
```

Значення заголовка `Authorization` і результат декодування:

```text
> Authorization: Basic dXNlcjpwYXNz
root@ns-se-201-01:/# echo 'dXNlcjpwYXNz' | base64 -d; echo
user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

```text
root@ns-se-201-01:/# curl -v 'https://www.se-201-01.lab:4443/reports/../items'
* Host www.se-201-01.lab:4443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:4443...
* connect to 10.53.0.81 port 4443 from 10.53.1.1 port 40022 failed: Connection refused
* Failed to connect to www.se-201-01.lab port 4443 after 1 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.se-201-01.lab port 4443 after 1 ms: Couldn't connect to server
```

Запит не вдався на етапі TCP-з'єднання. Ім'я розв'язалося (`Host www.se-201-01.lab:4443 was resolved`, `IPv4: 10.53.0.81`), але на рядку `connect to 10.53.0.81 port 4443 ... failed: Connection refused` з'єднання відхилили і далі вийшло `curl: (7) Failed to connect`. Отже, до TLS і до самого HTTP-запиту справа не дійшла, бо на порту 4443 сервера 10.53.0.81 ніхто не слухає.

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | `/items` | `/items` |
| `--path-as-is` | `/reports/../items` | `/items` |

Вивід двох запитів зі шляхом URI 4:

```text
root@ns-se-201-01:/# curl 'http://www.se-201-01.lab/reports/../items'
server: web-b
Host: www.se-201-01.lab
request URI: /items
path: /items
query:
client: 10.53.1.1
root@ns-se-201-01:/# curl --path-as-is 'http://www.se-201-01.lab/reports/../items'
server: web-b
Host: www.se-201-01.lab
request URI: /reports/../items
path: /items
query:
client: 10.53.1.1
```

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:

* URI 1: фрагмент `#section-2` curl не надсилає (у запиті `GET /reports/items?id=10&q=uk`), бо він потрібен тільки клієнту, тому `request URI` без нього.
* URI 2: `:80` зник із `Host: blog.se-201-01.lab`, бо 80 це стандартний порт для http, а `query` лишився закодованим (`id=%D0%BA%D0%B8%D1%97%D0%B2`), бо curl надіслав його як є.
* URI 3: `user:pass@` curl переніс в `Authorization: Basic dXNlcjpwYXNz` (це `user:pass` у base64), тому в `Host` його немає, а `#results` не надіслано, як і будь-який фрагмент.
* Шлях: звичайний curl сам стиснув `/reports/../items` до `/items`, а з `--path-as-is` надіслав `/reports/../items`, але сервер усе одно показав `path: /items`, тобто нормалізував шлях сам.

## Завдання 2. Проходження ієрархії вручну

```text
root@ns-se-201-01:/# dig @10.53.0.2 ns.se-201-01.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.2 ns.se-201-01.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27536
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: d59536af86be7e94010000006b0605a5c38e3199b2403e29 (good)
;; QUESTION SECTION:
;ns.se-201-01.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 81 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Tue Nov 24 21:46:13 UTC 2026
;; MSG SIZE  rcvd: 117

root@ns-se-201-01:/# dig @10.53.0.3 ns.se-201-01.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.3 ns.se-201-01.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14629
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: bb71ed6465d62973010000006b0605ab96ddf4b64ed5cc1c (good)
;; QUESTION SECTION:
;ns.se-201-01.lab.              IN      A

;; AUTHORITY SECTION:
se-201-01.lab.          3600    IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       3600    IN      A       10.53.1.1

;; Query time: 81 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Tue Nov 24 21:46:19 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-se-201-01:/# dig @10.53.1.1 ns.se-201-01.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 ns.se-201-01.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30131
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a666e0a35ffd23bc010000006b0605b09bd4c10a3154aa03 (good)
;; QUESTION SECTION:
;ns.se-201-01.lab.              IN      A

;; ANSWER SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 21:46:24 UTC 2026
;; MSG SIZE  rcvd: 103
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | `qr` | — (0) | `lab. 172800 IN NS ns1.nic.lab.` | `ns1.nic.lab. 172800 IN A 10.53.0.3` (+ OPT) |
| 10.53.0.3 | NOERROR | `qr` | — (0) | `se-201-01.lab. 3600 IN NS ns.se-201-01.lab.` | `ns.se-201-01.lab. 3600 IN A 10.53.1.1` (+ OPT) |
| 10.53.1.1 | NOERROR | `qr aa` | `ns.se-201-01.lab. 60 IN A 10.53.1.1` | `se-201-01.lab. 60 IN NS ns.se-201-01.lab.` | лише OPT |

Вивід `dig ns.se-201-01.lab A +trace`:

```text
root@ns-se-201-01:/# dig ns.se-201-01.lab A +trace

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> ns.se-201-01.lab A +trace
;; global options: +cmd
.                       85555   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

se-201-01.lab.          3600    IN      NS      ns.se-201-01.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 41 ms

ns.se-201-01.lab.       60      IN      A       10.53.1.1
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.se-201-01.lab) in 1 ms
```

Ланцюжок `+trace` збігається з ручним: root (10.53.0.2) → TLD `lab.` (10.53.0.3) → власний сервер (10.53.1.1); TTL і записи на кожному кроці ті самі.

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 se-201-01.lab NS +norec` | 3600 | немає (`flags: qr`) | AUTHORITY |
| `dig @10.53.1.1 se-201-01.lab NS +norec` | 60 | є (`flags: qr aa`) | ANSWER |

```text
root@ns-se-201-01:/# dig @10.53.0.3 se-201-01.lab NS +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.3 se-201-01.lab NS +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24612
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 733e856af0bc4ba0010000006b0605b9cdb6936be285c6b8 (good)
;; QUESTION SECTION:
;se-201-01.lab.                 IN      NS

;; AUTHORITY SECTION:
se-201-01.lab.          3600    IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       3600    IN      A       10.53.1.1

;; Query time: 40 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Tue Nov 24 21:46:33 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-se-201-01:/# dig @10.53.1.1 se-201-01.lab NS +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-201-01.lab NS +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17269
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a9a66fae64064ba6010000006b0605bf34b546628169d7e7 (good)
;; QUESTION SECTION:
;se-201-01.lab.                 IN      NS

;; ANSWER SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 21:46:39 UTC 2026
;; MSG SIZE  rcvd: 103
```

1. Які відповіді є відсиланнями і за якими ознаками це визначено?

   Відсиланнями є відповіді root (10.53.0.2) і TLD (10.53.0.3): `status: NOERROR`, у рядку flags немає `aa`, `ANSWER: 0`, а в секції AUTHORITY наведено NS-запис зони наступного рівня (`lab.` → `ns1.nic.lab.`, `se-201-01.lab.` → `ns.se-201-01.lab.`). Відповідь 10.53.1.1 — остаточна: прапорець `aa` і запис у секції ANSWER.

2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?

   `ns.se-201-01.lab. 3600 IN A 10.53.1.1` — це glue-запис: адреса сервера, на який делеговано зону. Ім'я сервера `ns.se-201-01.lab` лежить усередині самої делегованої зони, тому без glue резолвер мав би спитати адресу `ns.se-201-01.lab` у сервера зони `se-201-01.lab`, адресу якого він саме й шукає. Виникло б замкнене коло, і розв'язати ім'я було б неможливо.

3. Яка з двох копій запису NS є авторитативною і чому?

   Авторитативна копія — з власного сервера 10.53.1.1: у відповіді є прапорець `aa`, запис лежить у секції ANSWER, а його TTL 60 збігається з `$TTL` зони. Копія в TLD (TTL 3600, без `aa`, у секції AUTHORITY) — це дані делегування, які зберігаються в батьківській зоні `lab.` і лише вказують, куди звертатися.

## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```text
root@ns-se-201-01:/# zone-reload
zone se-201-01.lab/IN: loaded serial 2026112402
OK
zone reload queued
```

<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | є | `10.53.0.80` | 60 | так |
| `blog` CNAME | NOERROR | є | `www.se-201-01.lab.` | 60 | так |
| `mail` A | NOERROR | є | `10.53.0.80` | 60 | так |
| `@` MX | NOERROR | є | `25 mail.se-201-01.lab.` | 60 | так |
| `@` TXT | NOERROR | є | `"lab-token=3bc4ea6d"` | 60 | так |
| `edge` A | NOERROR | є | `10.53.0.81` | 50 | так |

Вивід шести запитів до власного сервера:

```text
root@ns-se-201-01:/# dig @10.53.1.1 www.se-201-01.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 www.se-201-01.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50820
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 2fcbc9bb7857ae05010000006b061c9f8809e75cbe32c01a (good)
;; QUESTION SECTION:
;www.se-201-01.lab.             IN      A

;; ANSWER SECTION:
www.se-201-01.lab.      60      IN      A       10.53.0.80

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:24:15 UTC 2026
;; MSG SIZE  rcvd: 123

root@ns-se-201-01:/# dig @10.53.1.1 blog.se-201-01.lab CNAME +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 blog.se-201-01.lab CNAME +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56136
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: adeea292fc247a78010000006b061cd1fbb2fd2ee99564b5 (good)
;; QUESTION SECTION:
;blog.se-201-01.lab.            IN      CNAME

;; ANSWER SECTION:
blog.se-201-01.lab.     60      IN      CNAME   www.se-201-01.lab.

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:25:05 UTC 2026
;; MSG SIZE  rcvd: 126

root@ns-se-201-01:/# dig @10.53.1.1 mail.se-201-01.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 mail.se-201-01.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38064
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e96f3f842a6e7e56010000006b061cd6eb64a537a7450732 (good)
;; QUESTION SECTION:
;mail.se-201-01.lab.            IN      A

;; ANSWER SECTION:
mail.se-201-01.lab.     60      IN      A       10.53.0.80

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:25:10 UTC 2026
;; MSG SIZE  rcvd: 124

root@ns-se-201-01:/# dig @10.53.1.1 se-201-01.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-201-01.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35062
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4cb5df6fcfd34b67010000006b061cda9a1f2510607471ad (good)
;; QUESTION SECTION:
;se-201-01.lab.                 IN      MX

;; ANSWER SECTION:
se-201-01.lab.          60      IN      MX      25 mail.se-201-01.lab.

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
mail.se-201-01.lab.     60      IN      A       10.53.0.80
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:25:14 UTC 2026
;; MSG SIZE  rcvd: 140

root@ns-se-201-01:/# dig @10.53.1.1 se-201-01.lab TXT +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-201-01.lab TXT +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 38199
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ebe12f561fa07159010000006b061ce4d6da918b50ac85b2 (good)
;; QUESTION SECTION:
;se-201-01.lab.                 IN      TXT

;; ANSWER SECTION:
se-201-01.lab.          60      IN      TXT     "lab-token=3bc4ea6d"

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:25:24 UTC 2026
;; MSG SIZE  rcvd: 134

root@ns-se-201-01:/# dig @10.53.1.1 edge.se-201-01.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 edge.se-201-01.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52638
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 14692d5b1380dc30010000006b061ce856666f72a187d4ca (good)
;; QUESTION SECTION:
;edge.se-201-01.lab.            IN      A

;; ANSWER SECTION:
edge.se-201-01.lab.     50      IN      A       10.53.0.81

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:25:28 UTC 2026
;; MSG SIZE  rcvd: 124
```

Порівняння відповідей резолвера та власного сервера:

```text
root@ns-se-201-01:/# dig blog.se-201-01.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> blog.se-201-01.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26029
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;blog.se-201-01.lab.            IN      A

;; ANSWER SECTION:
blog.se-201-01.lab.     60      IN      CNAME   www.se-201-01.lab.
www.se-201-01.lab.      60      IN      A       10.53.0.80

;; Query time: 40 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 24 23:26:22 UTC 2026
;; MSG SIZE  rcvd: 81

root@ns-se-201-01:/# dig @10.53.1.1 blog.se-201-01.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 blog.se-201-01.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 35725
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 5dea74ca3bd0266a010000006b061d22c50e1eee9cc0aeaf (good)
;; QUESTION SECTION:
;blog.se-201-01.lab.            IN      A

;; ANSWER SECTION:
blog.se-201-01.lab.     60      IN      CNAME   www.se-201-01.lab.
www.se-201-01.lab.      60      IN      A       10.53.0.80

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:26:26 UTC 2026
;; MSG SIZE  rcvd: 142

root@ns-se-201-01:/# dig se-201-01.lab MX

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> se-201-01.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7546
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;se-201-01.lab.                 IN      MX

;; ANSWER SECTION:
se-201-01.lab.          60      IN      MX      25 mail.se-201-01.lab.

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 24 23:26:30 UTC 2026
;; MSG SIZE  rcvd: 63

root@ns-se-201-01:/# dig @10.53.1.1 se-201-01.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 se-201-01.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8743
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: fc0403f3f0c634c0010000006b061d2a3b77506ae8334013 (good)
;; QUESTION SECTION:
;se-201-01.lab.                 IN      MX

;; ANSWER SECTION:
se-201-01.lab.          60      IN      MX      25 mail.se-201-01.lab.

;; AUTHORITY SECTION:
se-201-01.lab.          60      IN      NS      ns.se-201-01.lab.

;; ADDITIONAL SECTION:
mail.se-201-01.lab.     60      IN      A       10.53.0.80
ns.se-201-01.lab.       60      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:26:34 UTC 2026
;; MSG SIZE  rcvd: 140
```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| `blog` через resolver-a | `qr rd ra` | 2: CNAME, A | 0 | 1: OPT | 60, 60 |
| `blog` через 10.53.1.1 | `qr aa` | 2: CNAME, A | 1: NS | 2: A (`ns`), OPT | 60, 60 |
| MX через resolver-a | `qr rd ra` | 1: MX | 0 | 1: OPT | 60 |
| MX через 10.53.1.1 | `qr aa` | 1: MX | 1: NS | 3: A (`mail`), A (`ns`), OPT | 60 |

Пояснення відмінностей: відповідь резолвера (10.53.0.53) мала прапорці `qr rd ra`, а мого сервера (10.53.1.1) `qr aa`. `aa` (authoritative answer) означає, що сервер сам відповідальний за зону і дав відповідь зі свого файлу, а не з кешу, а `ra` (recursion available) означає, що резолвер вміє шукати відповідь за клієнта (`rd` це просто те, що клієнт просив рекурсію). У резолвера AUTHORITY і ADDITIONAL порожні (`AUTHORITY: 0`, `ADDITIONAL: 1`, а це лише OPT), бо він віддає готову відповідь, а мій сервер додає в AUTHORITY запис NS зони, а в ADDITIONAL адресу `ns.se-201-01.lab` (10.53.1.1), щоб клієнт одразу знав, хто відповідає за зону.

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
root@ns-se-201-01:/# zone-check
dns_master_load: /lab/zone/se-201-01.lab.zone:23: se-201-01.lab: CNAME and other data
zone se-201-01.lab/IN: loading from master file /lab/zone/se-201-01.lab.zone failed: CNAME and other data
zone se-201-01.lab/IN: not loaded due to errors.
```

1. Який TTL має кожен запис зони і звідки це значення взято?

   У файлі зони стоїть `$TTL 60`, тому `www`, `blog`, `mail`, MX, TXT, NS і `ns` мають TTL 60. `edge` має 50, бо там TTL указано явно (`edge 50 IN A`). SOA має 20, бо вказано в рядку `@ 20 IN SOA`.

2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?

   У ANSWER для `blog` два записи: CNAME `blog` → `www.se-201-01.lab.` і A `www.se-201-01.lab.` → `10.53.0.80`. Так сталося, бо `blog` це псевдонім, і разом з ним сервер одразу віддає адресу того імені, на яке він вказує, щоб не робити другий запит.

3. Чому запис CNAME неможливо розмістити для імені `@`?

   CNAME означає, що ім'я це просто псевдонім іншого імені, тому поруч з ним не може бути інших записів. Для `@` це неможливо, бо там уже є SOA, NS, MX і TXT, тому `zone-check` видав `CNAME and other data`, і зона не завантажилась (`not loaded due to errors`).

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text
root@ns-se-201-01:/# dig www.se-201-01.lab +noall +answer
www.se-201-01.lab.      60      IN      A       10.53.0.80
root@ns-se-201-01:/# dig www.se-201-01.lab +noall +answer
www.se-201-01.lab.      31      IN      A       10.53.0.80
```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:

Нова адреса з'явиться через resolver-a приблизно через R секунд після `zone-reload`, де R — залишок TTL запису в кеші resolver-a в момент зміни: `t_появи ≈ t_reload + R`. Найбільша можлива затримка дорівнює повному TTL запису, 60 с (`$TTL 60`), якщо резолвер закешував запис безпосередньо перед зміною. Додаткова похибка спостереження — до 2 с (цикл із `sleep 2`). *(Прогноз записано 25.11.2026 о 00:46 за київським часом, тобто до вимірювання о 23:50 UTC; формулювання підготовлено з допомогою ШІ, див. декларацію.)*

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
root@ns-se-201-01:/# zone-edit
root@ns-se-201-01:/# dig @10.53.1.1 www.se-201-01.lab +norec +short
10.53.0.80
root@ns-se-201-01:/# grep -E "serial|^www" /lab/zone/se-201-01.lab.zone
; After every change: increase the serial, then run  zone-reload
            2026112403 ; serial
www             IN A      10.53.0.81
root@ns-se-201-01:/#
root@ns-se-201-01:/#
```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```text
root@ns-se-201-01:/# date +%T; dig www.se-201-01.lab +noall +answer
23:50:28
www.se-201-01.lab.      60      IN      A       10.53.0.80
root@ns-se-201-01:/# zone-reload; date +%T
zone se-201-01.lab/IN: loaded serial 2026112403
OK
zone reload queued
23:51:06
root@ns-se-201-01:/# dig @10.53.1.1 www.se-201-01.lab +norec +short
10.53.0.81
```

Очікуваний час появи нової адреси (підставити R у формулу): запис потрапив у кеш о 23:50:28 з TTL 60, `zone-reload` виконано о 23:51:06, тобто через 38 с; на момент зміни R = 60 − 38 = 22 с, очікуваний час появи нової адреси: 23:51:06 + 22 с = **23:51:28**.

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text
root@ns-se-201-01:/# while true; do echo "$(date +%T) $(dig +short www.se-201-01.lab)"; sleep 2; done
23:51:19 10.53.0.80
23:51:21 10.53.0.80
23:51:23 10.53.0.80
23:51:25 10.53.0.80
23:51:27 10.53.0.80
23:51:29 10.53.0.81
^C
```

Рядок `server:` з відповіді `curl`: `server: web-b`

```text
root@ns-se-201-01:/# curl http://www.se-201-01.lab/
server: web-b
Host: www.se-201-01.lab
request URI: /
path: /
query:
client: 10.53.1.1
root@ns-se-201-01:/#
```

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 23:51:06 | 23:51:28 | 23:51:29 | 23 | 22 (R на момент зміни); найбільша можлива — 60 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?

   `zone-reload` закінчився о 23:51:06, а запис я закешував о 23:50:28, тому до кінця TTL лишалося 60 − 38 = 22 с, і нова адреса мала з'явитися о 23:51:28. Цикл перевіряв адресу раз на 2 с (`sleep 2`): остання перевірка зі старою адресою була о 23:51:27, наступна о 23:51:29, і там уже була нова `10.53.0.81`. Отже, 1 с різниці це похибка вимірювання через крок циклу 2 с, а не помилка прогнозу.

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?

   Найбільша затримка для цього запису дорівнює його TTL, тобто 60 с. Так буде, якщо резолвер закешував запис за мить до мого `zone-reload`: тоді кеш живе майже повні 60 с після зміни, і всі клієнти цього резолвера весь цей час отримують стару адресу.

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?

   Заздалегідь, не менше ніж за добу (старий TTL), треба зменшити TTL запису, наприклад до 60 с, щоб усі кеші встигли взяти коротке значення. Потім змінити адресу, і клієнти перейдуть на нову за хвилину, а після цього TTL можна повернути до великого. Старий сервер одразу вимикати не можна, бо частина клієнтів ще має стару адресу в кеші, тому його треба тримати працюючим, поки не мине час, на який записи ще можуть лежати в кеші.

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:

За RFC 2308 (§5) негативна відповідь NXDOMAIN кешується на час `min(TTL запису SOA, поле minimum SOA)`. Для варіанта: TTL запису SOA = 20 с, `minimum` = 90 с → min(20, 90) = **20 с**. Отже, після `zone-reload` resolver-a повертатиме NXDOMAIN для `new-wsrop.se-201-01.lab` ще не довше ніж ~20 с від моменту першого запиту, плюс до 2 с похибки циклу; TTL SOA в секції AUTHORITY очікується 20. *(Прогноз записано до вимірювання; формулювання підготовлено з допомогою ШІ, див. декларацію.)*

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text
root@ns-se-201-01:/# dig @10.53.1.1 new-wsrop.se-201-01.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.1.1 new-wsrop.se-201-01.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 56161
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6c2813824019abc9010000006b0623923c841c5ce3bd94f5 (good)
;; QUESTION SECTION:
;new-wsrop.se-201-01.lab.       IN      A

;; AUTHORITY SECTION:
se-201-01.lab.          20      IN      SOA     ns.se-201-01.lab. admin.se-201-01.lab. 2026112403 3600 600 86400 90

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Tue Nov 24 23:53:54 UTC 2026
;; MSG SIZE  rcvd: 125
```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text
root@ns-se-201-01:/# date +%T; dig new-wsrop.se-201-01.lab; zone-reload; date +%T
23:54:36

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> new-wsrop.se-201-01.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 19257
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-wsrop.se-201-01.lab.       IN      A

;; AUTHORITY SECTION:
se-201-01.lab.          20      IN      SOA     ns.se-201-01.lab. admin.se-201-01.lab. 2026112403 3600 600 86400 90

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Tue Nov 24 23:54:36 UTC 2026
;; MSG SIZE  rcvd: 97

zone se-201-01.lab/IN: loaded serial 2026112404
OK
zone reload queued
23:54:36
root@ns-se-201-01:/# dig @10.53.1.1 new-wsrop.se-201-01.lab +norec +short; while true; do echo "$(date +%T) $(dig new-wsrop.se-201-01.lab +noall +comments | grep -o 'status: [A-Z]*')"; sleep 2; done
10.53.0.80
```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text
root@ns-se-201-01:/# dig @10.53.1.1 new-wsrop.se-201-01.lab +norec +short; while true; do echo "$(date +%T) $(dig new-wsrop.se-201-01.lab +noall +comments | grep -o 'status: [A-Z]*')"; sleep 2; done
23:54:39 status: NXDOMAIN
23:54:41 status: NXDOMAIN
23:54:43 status: NXDOMAIN
23:54:45 status: NXDOMAIN
23:54:47 status: NXDOMAIN
23:54:49 status: NXDOMAIN
23:54:51 status: NXDOMAIN
23:54:53 status: NXDOMAIN
23:54:55 status: NXDOMAIN
23:54:57 status: NOERROR
23:54:59 status: NOERROR
^C
root@ns-se-201-01:/#
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 23:54:36 | 23:54:36 | 23:54:57 | ≈ 21 (останній NXDOMAIN о 23:54:55, перший NOERROR о 23:54:57) | 20 |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?

   Негативна відповідь означає, що запису запитаного імені немає, тому в неї нема свого TTL, за яким її можна було б кешувати. Сервер тому кладе в секцію AUTHORITY запис SOA зони (`se-201-01.lab. 20 IN SOA ...`), і резолвер кешує саме відсутність імені на час, який береться з цього SOA.

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

   Так, збігається: у файлі зони `@ 20 IN SOA` (TTL 20), а в AUTHORITY відповіді резолвера `se-201-01.lab. 20 IN SOA`. За RFC 2308 береться менше з двох чисел, TTL запису SOA (20) і `minimum` (90), тому негативний кеш живе 20 с.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

   Перша перевірка до створення покаже `NXDOMAIN`, і резолвер запам'ятає це на 20 с. Друга перевірка одразу після створення теж покаже `NXDOMAIN` з його кешу, хоча запис уже є, і скрипт вирішить, що розгортання не вдалося. У моєму досліді з `new-wsrop` так і було: від 23:54:39 до 23:54:55 резолвер відповідав `NXDOMAIN`, і лише о 23:54:57 з'явився `NOERROR`. Виправити можна так: після створення питати напряму авторитативний сервер (`dig @10.53.1.1 ... +norec`, він одразу віддав `10.53.0.80`), або чекати 20 с, або не питати про ім'я до створення.

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | лише власний сервер 10.53.1.1 (делегування `lab.` і `se-201-01.lab` уже в кеші) → ≈ 1 мс | нікуди, негативна відповідь з кешу → ≈ 0 мс |
| B: нове ім'я в зоні `lab.` | лише TLD 10.53.0.3 → ≈ 40 мс | нікуди, з кешу → ≈ 0 мс |
| C: ім'я в неіснуючому TLD | лише root 10.53.0.2 → ≈ 80 мс | нікуди, з кешу → ≈ 0 мс |
| D: `www.se-201-01.lab` після перезапуску резолвера | root → TLD → власний сервер (кеш порожній) → ≈ 80 + 40 + 1 ≈ 120 мс | нікуди, з кешу → ≈ 0 мс |

*(Прогноз записано до вимірювання на основі `+trace` із завдання 2: root ≈ 80 мс, TLD ≈ 41 мс, власний сервер ≈ 1 мс; формулювання підготовлено з допомогою ШІ, див. декларацію.)*

Вивід вимірювань:

```text
root@ns-se-201-01:/# A=y$RANDOM.se-201-01.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 0 msec
;; Query time: 0 msec
root@ns-se-201-01:/#
root@ns-se-201-01:/# B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 39 msec
;; Query time: 1 msec
root@ns-se-201-01:/# C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 79 msec
;; Query time: 1 msec
root@ns-se-201-01:/#

# --- Рядок D. Термінал codespace (хост):
@Yurchuk19 ➜ /workspaces/opism-lab-DNS (master) $ docker compose -f stand/compose.yaml restart resolver-a
[+] restart 0/1
 ⠸ Container dnslab-resolver-a-1 Restarting                                         0.4s
# --- Термінал стенда:
root@ns-se-201-01:/# dig www.se-201-01.lab | grep 'Query time'; dig www.se-201-01.lab | grep 'Query time'
;; Query time: 401 msec
;; Query time: 0 msec
# --- Термінал codespace (хост):
@Yurchuk19 ➜ /workspaces/opism-lab-DNS (master) $ docker compose -f stand/compose.yaml restart resolver-a
[+] restart 0/1
 ⠹ Container dnslab-resolver-a-1 Restarting                                         0.3s
@Yurchuk19 ➜ /workspaces/opism-lab-DNS (master) $
# --- Термінал стенда:
root@ns-se-201-01:/# curl -so /dev/null -w '%{time_namelookup}\n' http://www.se-201-01.lab/
0.402504
root@ns-se-201-01:/#
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 0 | 0 | лише власний сервер (у журналі власного сервера `y17642.se-201-01.lab` о 23:56:11; у журналах root і TLD цього імені немає) |
| B | 39 | 1 | лише TLD (`x13824.lab` у журналі TLD о 23:56:17) |
| C | 79 | 1 | лише root (`x1330.zz13843` у журналі root о 23:56:22) |
| D | 401 | 0 | root тричі (`. NS`, `a.root-servers.net A`, `www.se-201-01.lab A`), потім TLD, потім власний сервер (23:58:47) |

Значення `time_namelookup` з `curl` після перезапуску резолвера: `0.402504` с

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text
# root (docker compose -f stand/compose.yaml exec root tail -n 30 /var/log/named/queries.log):
24-Nov-2026 23:58:47.393 client @0x7ad4e8004488 10.53.0.53#40034 (.): query: . IN NS -E(0)DC (10.53.0.2)
24-Nov-2026 23:58:47.473 client @0x7ad4e8004488 10.53.0.53#6423 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
24-Nov-2026 23:58:47.554 client @0x7ad4e8004488 10.53.0.53#35926 (www.se-201-01.lab): query: www.se-201-01.lab IN A -E(0)DC (10.53.0.2)
# TLD (docker compose -f stand/compose.yaml exec tld tail -n 30 /var/log/named/queries.log):
24-Nov-2026 23:58:47.675 client @0x7da720004488 10.53.0.53#18396 (www.se-201-01.lab): query: www.se-201-01.lab IN A -E(0)DC (10.53.0.3)
# власний сервер (querylog):
24-Nov-2026 23:58:47.715 client @0x7bf8e006ac68 10.53.0.53#37815 (www.se-201-01.lab): query: www.se-201-01.lab IN A -E(0)DC (10.53.1.1)
```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.

   З перших запитів у таблиці: наш сервер дає майже 0 мс (рядок A), TLD додає близько 39 мс (рядок B), root близько 79 мс (рядок C). Так видно, бо кожен рядок зачіпає рівно один сервер: у A резолвер питає тільки наш сервер, у B тільки TLD для `.lab`, у C тільки root, бо зони `zz` не існує. Другі запити в усіх рядках (0 і 1 мс) беруться вже з кешу резолвера.

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?

   У рядку A резолвер уже знає з попередніх завдань, де знаходиться сервер зони `se-201-01.lab` (запис NS і адресу `ns.se-201-01.lab`, 10.53.1.1), і де сервер зони `lab.` (`ns1.nic.lab`). Тому він іде одразу до нашого сервера, а root і TLD не питає. Це видно й у журналах: запит `y17642.se-201-01.lab` (23:56:11) є тільки в журналі нашого сервера, а в журналах root і TLD його немає.

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?

   Кожен з п'яти доменів потребує окремого пошуку імені, якого ще немає в кеші, а в моєму рядку D порожній кеш дав 401 мс на один пошук. Тому під час першого відкриття сторінка чекає на п'ять таких пошуків, і це додає помітну затримку, навіть якщо браузер робить їх паралельно. Під час повторного відкриття всі п'ять імен уже в кеші (у рядку D другий запит 0 мс), тому ця затримка майже зникає.

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:

- **resolver-a (звичайний):** запис у кеші вже сплив, сервер зони не відповідає → резолвер намагається достукатися до 10.53.1.1, повторює спроби і після тайм-ауту повертає **SERVFAIL**; відповідь буде повільною (секунди, а не мілісекунди).
- **resolver-b (serve-stale, RFC 8767):** повертає застарілу відповідь `10.53.0.81` з кешу з малим TTL (кілька десятків секунд), щоб клієнт скоро перепитав; відповідь відносно швидка.
- Після `named-start` resolver-a почне знову віддавати адресу не миттєво, а з невеликою затримкою.

*(Прогноз записано до вимірювання; формулювання підготовлено з допомогою ШІ, див. декларацію.)*

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text
root@ns-se-201-01:/# dig @10.53.0.53 edge.se-201-01.lab +noall +answer; dig @10.53.0.54 edge.se-201-01.lab +noall +answer
edge.se-201-01.lab.     50      IN      A       10.53.0.81
edge.se-201-01.lab.     50      IN      A       10.53.0.81
root@ns-se-201-01:/# named-stop; date +%T; sleep 60; date +%T; time dig @10.53.0.53 edge.se-201-01.lab +tries=1 +timeout=30; time dig @10.53.0.53 edge.se-201-01.lab +tries=1 +timeout=30; time dig @10.53.0.54 edge.se-201-01.lab +tries=1 +timeout=30
00:03:58
00:04:58
```

Запити із заміром часу (крок 5):

```text
root@ns-se-201-01:/# named-stop; date +%T; sleep 60; date +%T; time dig @10.53.0.53 edge.se-201-01.lab +tries=1 +timeout=30; time dig @10.53.0.53 edge.se-201-01.lab +tries=1 +timeout=30; time dig @10.53.0.54 edge.se-201-01.lab +tries=1 +timeout=30
00:03:58
00:04:58
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.53 edge.se-201-01.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.026s
user    0m0.009s
sys     0m0.002s

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.53 edge.se-201-01.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 5167
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;edge.se-201-01.lab.            IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Wed Nov 25 00:05:28 UTC 2026
;; MSG SIZE  rcvd: 47


real    0m0.021s
user    0m0.005s
sys     0m0.005s

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @10.53.0.54 edge.se-201-01.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29573
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;edge.se-201-01.lab.            IN      A

;; ANSWER SECTION:
edge.se-201-01.lab.     30      IN      A       10.53.0.81

;; Query time: 1799 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Wed Nov 25 00:05:30 UTC 2026
;; MSG SIZE  rcvd: 63


real    0m1.819s
user    0m0.003s
sys     0m0.006s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | тайм-аут (`no servers could be reached`) | — | 0m30.026s |
| resolver-a, другий | SERVFAIL | — | 0m0.021s |
| resolver-b | NOERROR | `edge.se-201-01.lab. 30 IN A 10.53.0.81` | 0m1.819s |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text
root@ns-se-201-01:/# named-start; date +%T; while true; do echo "$(date +%T) $(dig @10.53.0.53 edge.se-201-01.lab +short +tries=1 +timeout=3)"; sleep 2; done
00:05:47
00:05:47
00:05:49
00:05:51
00:05:53
00:05:55
00:05:57
00:05:59 10.53.0.81
^C
```

Перевірка після завдання (`dig @10.53.1.1 se-201-01.lab SOA +norec +short`):

```text
root@ns-se-201-01:/# dig @10.53.1.1 se-201-01.lab SOA +norec +short
ns.se-201-01.lab. admin.se-201-01.lab. 2026112404 3600 600 86400 90
root@ns-se-201-01:/#
```

### Постмортем

1. Що сталося: мій авторитативний сервер було зупинено командою `named-stop` о 00:03:58, і він не працював до `named-start` о 00:05:47, тобто приблизно 1 хв 49 с.
2. Вплив на клієнтів resolver-a і resolver-b: через resolver-a перший запит 30 с не мав відповіді, а потім прийшов `SERVFAIL`, тобто ім'я не розв'язувалось. Через resolver-b за 1,8 с прийшла стара адреса `10.53.0.81` з TTL 30.
3. Хронологія з виміряним часом: 00:03:58 зупинка, 00:04:58 TTL запису в кеші сплив, 00:05:28 `SERVFAIL` від resolver-a, 00:05:30 resolver-b віддав старий запис, 00:05:47 запуск сервера, 00:05:59 resolver-a знову віддає адресу.
4. Чому кеш допоміг або не допоміг: у resolver-a запис сплив, а прострочені дані він не віддає, тому клієнти лишились без відповіді. Resolver-b має serve-stale, тому віддав застарілий запис, і для клієнтів сервіс працював.
5. Що слід змінити: додати другий авторитативний сервер (ще один NS), щоб зона не залежала від одного; увімкнути serve-stale на резолверах; поставити більший TTL для важливих записів, щоб кеш пережив коротку аварію; налаштувати моніторинг, щоб дізнаватись про падіння за хвилини.

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

   Запис мав TTL 30, хоча у файлі зони було 50. Короткий TTL для застарілих даних потрібен, щоб клієнт не тримав їх довго у своєму кеші і скоро спитав знову, і коли сервер повернеться, дані швидко оновляться.

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

   Мабуть, перший запит зайняв 30 с, бо резолвер чекав на відповідь від мертвого сервера, і після цього запам'ятав, що сервер недоступний. Тому другий запит він одразу завершив `SERVFAIL`, не намагаючись знову (0 мс).

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

   Я думаю, що resolver-a ще якийсь час вважав мій сервер недоступним і давав `SERVFAIL` (у `+short` це порожній рядок), а перевірив його знову лише приблизно через 12 с (00:05:47 → 00:05:59). Точно визначити це з мого виводу не можна.

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

   Serve-stale шкодить, коли дані змінилися, а резолвер про це не знає. Якби під час зміни адреси `www` (завдання 4) авторитативний сервер упав, резолвер віддавав би стару адресу `10.53.0.80`, і клієнти ходили б на старий сервер, поки авторитативний сервер не повернеться, а не лише до кінця TTL.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text
; Zone se-201-01.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 60
@     20 IN SOA ns.se-201-01.lab. admin.se-201-01.lab. (
            2026112404 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            90         ; minimum
        )
@     IN NS ns.se-201-01.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www             IN A      10.53.0.81
blog            IN CNAME  www
mail            IN A      10.53.0.80
@               IN MX     25 mail
@               IN TXT    "lab-token=3bc4ea6d"
edge       50   IN A      10.53.0.81



new-wsrop       IN A      10.53.0.80
```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

Найбільш несподіваним для мене був результат рядка D завдання 6. За рядками B і C таблиці (39 мс на TLD і 79 мс на root) я чекав приблизно 120 мс, а після перезапуску resolver-a вийшло `;; Query time: 401 msec`, тобто втричі більше. Журнал root пояснює частину цього: резолвер звернувся до нього не один раз, а тричі, `. IN NS` о 23:58:47.393, `a.root-servers.net IN A` о 23:58:47.473 і `www.se-201-01.lab IN A` о 23:58:47.554. Після перезапуску кеш був порожній, тому резолверу спочатку довелося дізнатися, хто такий root, і тільки потім питати про моє ім'я.

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:

Без відповіді лишилося питання, куди поділось приблизно 80 мс із 401. Від першого запису в журналі root (23:58:47.393) до запиту в журналі мого сервера (23:58:47.715) минуло близько 320 мс, а виміряно 401. Я перевірив журнали root, TLD і мого сервера, і в них цей проміжок не видно. З'ясувати не вдалося, що резолвер робив до першого запиту в журналі root і після останнього запиту до мого сервера, бо журналів самого резолвера я не бачив.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

Примітка до таблиці. Промпти наведено дослівно. Повідомлення, які містили лише вставлений вивід терміналу або скріншот, позначено як «[вивід терміналу …]» / «[скріншот …]»: самі виводи повністю наведено у звіті. Повідомлення з текстами моїх відповідей позначено як «[мій текст …]»: ці тексти наведено у звіті дослівно, з виправленнями ШІ, вказаними в останній колонці.


| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | Claude Opus 5.5 | [скріншоти терміналу: up.sh, вхід у стенд, редактор nano, zone-reload] | ШІ пояснював кожен крок|
| 2 | Claude Opus 5.5 | [вивід терміналу завдання 3] | ШІ перевірив записи |
