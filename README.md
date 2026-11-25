# Практична робота № 1

**Дисципліна:** Основи побудови інформаційних систем та мереж

**Тема:** Спостереження за процесом звернення до вебресурсу. Побудова власної моделі рівнів взаємодії

| | |
|---|---|
| **Прізвище, ім'я** | Юрчук Дмитро |
| **Група** | ЗКІ-21  |
| **Номер варіанта** | 1 |
| **Домен варіанта** | `example.com` |
| **Середовище виконання** | Windows 11, PowerShell 5.1 |
| **Версія curl** | curl 8.22.0 (Windows) |
| **Дата виконання** | 2026-11-24 |

---

## Частина A. Збір експериментальних даних

### A.1. Запит із діагностичним виводом

**Команда:**

```
curl.exe -v https://example.com
```

**Вивід:**

```
* Host example.com:443 was resolved.
* IPv6: (none)
* IPv4: 172.66.147.243, 104.20.23.154
*   Trying 172.66.147.243:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* ALPN: server accepted http/1.1
* Established connection to example.com (172.66.147.243 port 443) from 192.168.179.204 port 54739 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed

  0      0   0      0   0      0      0      0                              0* using HTTP/1.x
> GET / HTTP/1.1

> Host: example.com

> User-Agent: curl/8.22.0

> Accept: */*

> 

* Request completely sent off
* schannel: remote party requests renegotiation
* schannel: renegotiating SSL/TLS connection
* schannel: SSL/TLS connection renegotiated
< HTTP/1.1 200 OK

< Date: Tue, 24 Nov 2026 17:28:41 GMT

< Content-Type: text/html; charset=utf-8

< Transfer-Encoding: chunked

< Connection: keep-alive

< Server: cloudflare

< last-modified: Thu, 19 Nov 2026 22:11:52 GMT

< allow: GET, HEAD

< Accept-Ranges: bytes

< Age: 12959

< cf-cache-status: HIT

< CF-RAY: a5fa21adea417581-DUS

< alt-svc: h3=":443"; ma=86400

< 

{ [584 bytes data]

100    577   0    577   0      0    989      0                              0
100    577   0    577   0      0    989      0                              0
100    577   0    577   0      0    988      0                              0
* Connection #0 to host example.com:443 left intact
<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content="width=device-width,initial-scale=1"><title>Example Domain</title><style>html{color-scheme:light dark;background:light-dark(#eee,#222)}body{font:16px/1.6 system-ui,sans-serif;max-width:26em;margin:auto;padding:25vh 2em 2em;text-align:center}</style></head><body><p>This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.</p><script src=/s.js></script></body></html>
```

---

### A.2. Запит без захисту з'єднання

**Команда:**

```
curl.exe -v http://neverssl.com
```

**Вивід:**

```
* Host neverssl.com:80 was resolved.
* IPv6: (none)
* IPv4: 34.223.124.45
*   Trying 34.223.124.45:80...
* Established connection to neverssl.com (34.223.124.45 port 80) from 192.168.179.204 port 54741 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed

  0      0   0      0   0      0      0      0           00:15              0* using HTTP/1.x
> GET / HTTP/1.1

> Host: neverssl.com

> User-Agent: curl/8.22.0

> Accept: */*

> 

* Request completely sent off
< HTTP/1.1 200 OK

< Date: Tue, 24 Nov 2026 17:28:57 GMT

< Server: Apache/2.4.68 ()

< Upgrade: h2,h2c

< Connection: Upgrade

< Last-Modified: Wed, 29 Jun 2022 00:23:33 GMT

< ETag: "f79-5e28b29d38e93"

< Accept-Ranges: bytes

< Content-Length: 3961

< Vary: Accept-Encoding

< Content-Type: text/html; charset=UTF-8

< 

{ [1000 bytes data]

100   3961 100   3961   0      0    252      0   00:15   00:15              0
100   3961 100   3961   0      0    252      0   00:15   00:15              0
100   3961 100   3961   0      0    252      0   00:15   00:15              0
* Connection #0 to host neverssl.com:80 left intact
<html>
	<head>
		<title>NeverSSL - Connecting ... </title>
		<style>
		body {
			font-family: Montserrat, helvetica, arial, sans-serif;
			font-size: 16x;
			color: #444444;
			margin: 0;
		}
		h2 {
			font-weight: 700;
			font-size: 1.6em;
			margin-top: 30px;
		}
		p {
			line-height: 1.6em;
		}
		.container {
			max-width: 650px;
			margin: 20px auto 20px auto;
			padding-left: 15px;
			padding-right: 15px
		}
		.header {
			background-color: #42C0FD;
			color: #FFFFFF;
			padding: 10px 0 10px 0;
			font-size: 2.2em;
		}
		.notice {
			background-color: red;
			color: white;
			padding: 10px 0 10px 0;
			font-size: 1.25em;
			animation: flash 4s infinite;
		}
		@keyframes flash {
		0% {
			background-color: red;
		}
		50% {
			background-color: #AA0000;
		}
		0% {
			background-color: red;
		}
		}
		<!-- CSS from Mark Webster https://gist.github.com/markcwebster/9bdf30655cdd5279bad13993ac87c85d -->
		</style>

		<script>
			var adjectives = [ 'cool' , 'calm' , 'relaxed', 'soothing', 'serene', 'slow',
							'beautiful', 'wonderful', 'wonderous', 'fun', 'good',
							'glowing', 'inner', 'grand', 'majestic', 'astounding',
							'fine', 'splendid', 'transcendent', 'sublime', 'whole',
							'unique', 'old', 'young', 'fresh', 'clear', 'shiny',
							'shining', 'lush', 'quiet', 'bright', 'silver' ];

			var nouns =	  [ 'day', 'dawn', 'peace', 'smile', 'love', 'zen', 'laugh',
							'yawn', 'poem', 'song', 'joke', 'verse', 'kiss', 'sunrise',
							'sunset', 'eclipse', 'moon', 'rainbow', 'rain', 'plan',
							'play', 'chart', 'birds', 'stars', 'pathway', 'secret',
							'treasure', 'melody', 'magic', 'spell', 'light', 'morning'];

			var prefix =
					// Choose 3 zen adjectives
					adjectives.sort(function(){return 0.5-Math.random()}).slice(-3).join('')
					+
					// Coupled with a zen noun
					nouns.sort(function(){return 0.5-Math.random()}).slice(-1).join('');
			window.location.href = 'http://' + prefix + '.neverssl.com/online';
		</script>
	</head>
	<body>
	<noscript>
		<div class="notice">
			<div class="container">
				⚠️ JavaScript appears to be disabled. NeverSSL's cache-busting works better if you enable JavaScript for <code>neverssl.com</code>.
			</div>
		</div>
	</noscript>
	<div class="header">
		<div class="container">
		<h1>NeverSSL</h1>
		</div>
	</div>
	<div class="content">
	<div class="container">

	<h1 id="status"></h1>
	<script>document.querySelector("#status").textContent = "Connecting ...";</script>
	<noscript>

		<h2>What?</h2>
		<p>This website is for when you try to open Facebook, Google, Amazon, etc
		on a wifi network, and nothing happens. Type "http://neverssl.com"
		into your browser's url bar, and you'll be able to log on.</p>

		<h2>How?</h2>
		<p>neverssl.com will never use SSL (also known as TLS). No
		encryption, no strong authentication, no <a
		href="https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security">HSTS</a>,
		no HTTP/2.0, just plain old unencrypted HTTP and forever stuck in the dark
		ages of internet security.</p>

		<h2>Why?</h2>
		<p>Normally, that's a bad idea. You should always use SSL and secure
		encryption when possible. In fact, it's such a bad idea that most websites
		are now using https by default.</p>

		<p>And that's great, but it also means that if you're relying on
		poorly-behaved wifi networks, it can be hard to get online.  Secure
		browsers and websites using https make it impossible for those wifi
		networks to send you to a login or payment page. Basically, those networks
		can't tap into your connection just like attackers can't. Modern browsers
		are so good that they can remember when a website supports encryption and
		even if you type in the website name, they'll use https.</p>

		<p>And if the network never redirects you to this page, well as you can
		see, you're not missing much.</p>

        <a href="https://twitter.com/neverssl">Follow @neverssl</a>

	</noscript>

	</div>
	</div>

	</body>
</html>
```

---

### A.3. Запит до служби доменних імен

*Виконано у Windows: `Resolve-DnsName` замість `dig` (п. 3.1 методички).*

**Команда (перше виконання):**

```
Resolve-DnsName example.com
```

**Вивід:**

```
# виконано: 2026-11-24 18:29:04

Name        Type TTL Section IPAddress              
----        ---- --- ------- ---------              
example.com AAAA 156 Answer  2606:4700:10::6814:179a
example.com AAAA 156 Answer  2606:4700:10::ac42:93f3
example.com A    252 Answer  104.20.23.154          
example.com A    252 Answer  172.66.147.243         
```

**Команда (повторне виконання через 5–7 хвилин):**

```
Resolve-DnsName example.com
```

**Вивід:**

```
# виконано: 2026-11-24 18:35:05

Name        Type TTL Section IPAddress              
----        ---- --- ------- ---------              
example.com AAAA 158 Answer  2606:4700:10::ac42:93f3
example.com AAAA 158 Answer  2606:4700:10::6814:179a
example.com A    217 Answer  172.66.147.243         
example.com A    217 Answer  104.20.23.154          
```

**Зафіксовані значення:**

| Параметр | Перше виконання | Повторне виконання |
|---|---|---|
| Час виконання (год:хв) | 18:29 | 18:35 |
| IP-адреса | 104.20.23.154, 172.66.147.243 | 172.66.147.243, 104.20.23.154 |
| Значення TTL | 252 | 217 |

> Якщо друге значення TTL виявилося більшим за перше — це нормально: кеш резолвера встиг оновитися. Зафіксуйте як є.

---

### A.4. Контрольний ресурс

**Команда:**

```
curl.exe -v https://google.com
```

**Вивід:**

```
* Host google.com:443 was resolved.
* IPv6: (none)
* IPv4: 142.250.154.101, 142.250.154.138, 142.250.154.113, 142.250.154.139, 142.250.154.100, 142.250.154.102
*   Trying 142.250.154.101:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* ALPN: server accepted http/1.1
* Established connection to google.com (142.250.154.101 port 443) from 192.168.179.204 port 54742 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed

  0      0   0      0   0      0      0      0                              0* using HTTP/1.x
> GET / HTTP/1.1

> Host: google.com

> User-Agent: curl/8.22.0

> Accept: */*

> 

* Request completely sent off
* schannel: remote party requests renegotiation
* schannel: renegotiating SSL/TLS connection
* schannel: SSL/TLS connection renegotiated
< HTTP/1.1 301 Moved Permanently

< Location: https://www.google.com/

< Content-Type: text/html; charset=UTF-8

< Content-Security-Policy-Report-Only: object-src 'none';base-uri 'self';script-src 'nonce-zWruh5LW-HtOyVhtYjSBpw' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp

< Date: Tue, 24 Nov 2026 17:28:58 GMT

< Expires: Thu, 24 Dec 2026 17:28:58 GMT

< Cache-Control: public, max-age=2592000

< Server: gws

< Content-Length: 220

< X-XSS-Protection: 0

< X-Frame-Options: SAMEORIGIN

< Alt-Svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

< 

{ [220 bytes data]

100    220 100    220   0      0    681      0                              0
100    220 100    220   0      0    681      0                              0
100    220 100    220   0      0    680      0                              0
* Connection #0 to host google.com:443 left intact
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
```

---

### A.5. Ресурси з некоректною конфігурацією сертифіката

**Випадок 1**

```
curl.exe -v https://expired.badssl.com
```

```
* Host expired.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
*   Trying 104.154.89.105:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* schannel: next InitializeSecurityContext failed: SEC_E_CERT_EXPIRED (0x80090328) - The received certificate has expired.
* closing connection #0
curl: (35) schannel: next InitializeSecurityContext failed: SEC_E_CERT_EXPIRED (0x80090328) - The received certificate has expired.
```

**Випадок 2**

```
curl.exe -v https://wrong.host.badssl.com
```

```
*   Trying 104.154.89.105:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* Host wrong.host.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
* schannel: SNI or certificate check failed: SEC_E_WRONG_PRINCIPAL (0x80090322) - The target principal name is incorrect.
* closing connection #0
curl: (60) schannel: SNI or certificate check failed: SEC_E_WRONG_PRINCIPAL (0x80090322) - The target principal name is incorrect.
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the webpage mentioned above.
```

**Випадок 3**

```
curl.exe -v https://self-signed.badssl.com
```

```
* Host self-signed.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
*   Trying 104.154.89.105:443...
* schannel: disabled automatic use of client certificate
* ALPN: curl offers http/1.1
* schannel: SEC_E_UNTRUSTED_ROOT (0x80090325) - The certificate chain was issued by an authority that is not trusted.
* closing connection #0
curl: (60) schannel: SEC_E_UNTRUSTED_ROOT (0x80090325) - The certificate chain was issued by an authority that is not trusted.
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the webpage mentioned above.
```

> Якщо використано альтернативний спосіб із параметром `--resolve` — зазначити це та навести фактичну команду.

---

## Частина B. Власна модель рівнів

**Кількість виділених груп:** 5

| № | Назва групи (власне формулювання) | Рядки виводу, віднесені до групи | Обґрунтування |
|---|---|---|---|
| 1 | Вміст сторінки | `{ [584 bytes data]`; HTML-код `<!doctype html>…` | Саме те, що бачить користувач; решта рядків потрібна лише для того, щоб це отримати |
| 2 | Запит і відповідь HTTP | `> GET / HTTP/1.1`, `> Host: example.com`, `> User-Agent: curl/8.22.0`, `> Accept: */*`, `* Request completely sent off`, `< HTTP/1.1 200 OK` і всі наступні рядки `<` до порожнього | Текстові повідомлення «що просить клієнт — що відповідає сервер»; однакові за формою в A.1 і A.2 |
| 3 | Захищений канал | `* ALPN: curl offers http/1.1`, `* ALPN: server accepted http/1.1`, `* schannel: remote party requests renegotiation`, `* schannel: renegotiating SSL/TLS connection`, `* schannel: SSL/TLS connection renegotiated` | Ці рядки є в A.1 (`https://`) і повністю відсутні в A.2 (`http://`), тобто це окремий шар між з'єднанням і HTTP |
| 4 | З'єднання з адресою й портом | `*   Trying 172.66.147.243:443...`, `* Established connection to example.com (172.66.147.243 port 443) from 192.168.179.204 port 54739`, `* Connection #0 to host example.com:443 left intact` | Оперують лише IP-адресами й номерами портів обох сторін; без цього кроку не може початися жоден наступний |
| 5 | Пошук адреси за іменем | `* Host example.com:443 was resolved.`, `* IPv6: (none)`, `* IPv4: 172.66.147.243, 104.20.23.154` | Перетворення імені на IP-адресу; відбувається першим, до будь-якого звернення до сервера |

*Групи впорядковано від найближчої до користувача (№ 1) до найближчої до апаратного забезпечення.*

**Рядки, які не вдалося віднести до жодної групи:**

| Рядок виводу | Причина утруднення |
|---|---|
| `* schannel: disabled automatic use of client certificate` | Стосується захищеного каналу, але описує налаштування клієнта, а не крок взаємодії з сервером |
| `< alt-svc: h3=":443"; ma=86400` | Формально поле відповіді HTTP (група 2), але за змістом пропонує інший спосіб з'єднання, тобто стосується нижчих груп |
| `* Connection #0 to host example.com:443 left intact` | Віднесено до групи 4, але рядок з'являється після обміну HTTP і говорить про повторне використання з'єднання, яке керується з рівня HTTP (`Connection: keep-alive`) |

---

## Контрольні питання

**1. Скільки рядків діагностичного виводу передує отриманню даних сторінки (завдання A.1)?**

> До першого рядка HTML у виводі A.1 ідуть 33 рядки діагностики. Я рахував усі рядки, що починаються з `*`, `>`, `<` або `{`.

**2. Які рядки наявні у виводі A.1 і відсутні у виводі A.2? Чим це зумовлено?**

> У A.1 є рядки про ALPN і три рядки `schannel: ... renegotiat...`, а в A.2 їх немає, бо `https://` вимагає TLS-рукостискання між TCP-підключенням і HTTP-запитом, а `http://` передає запит одразу, без шифрування. Рядки ALPN з'являються саме під час рукостискання, там клієнт і сервер домовляються про протокол. Рядки `schannel: ... renegotiat...` стоять уже після `Request completely sent off`, тобто сервер попросив ще раз узгодити параметри TLS, коли запит уже був надісланий, і тільки потім прийшла відповідь `HTTP/1.1 200 OK`.

**3. Звідки у виводі з'явилося значення `443`, якщо його не було вказано в адресі?**

> Порт 443 curl бере за замовчуванням для схеми `https://`, а порт 80 для `http://`, тому порт з'являється у виводі, хоча в адресі його не вказано.

**4. Як змінилося значення TTL між двома запитами (A.3)? Що означає це число?**

> TTL зменшився з 252 до 217, тобто на 35 с, хоча минуло 360 с. Отже, за 360 с запис у кеші вже спливав (252 − 360 < 0), і резолвер після цього заново запитав авторитативний сервер та отримав запис з новим відліком. Тому 217 — це залишок часу вже оновленого запису, а не продовження старого відліку. TTL (time to live) означає, скільки секунд резолвер може зберігати DNS-відповідь у кеші, перш ніж запитати її знову.

**5. Чим відрізняються між собою три причини помилок із завдання A.5? Сформулювати кожну однією фразою.**

| Випадок | Причина недовіри |
|---|---|
| `expired` | (`SEC_E_CERT_EXPIRED`) термін дії сертифіката закінчився, тобто проблема в часі |
| `wrong.host` | (`SEC_E_WRONG_PRINCIPAL`) ім'я в сертифікаті не збігається з іменем хоста, до якого підключаємося, тобто проблема в імені |
| `self-signed` | (`SEC_E_UNTRUSTED_ROOT`) сертифікат видав сам власник, а не довірений центр сертифікації, тому ланцюжок довіри закінчується на корені, якому система не довіряє |

**6. Три рядки з власних виводів, про які не йшлося на лекції 1:**

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | `< cf-cache-status: HIT` | A.1 |
| 2 | `< Upgrade: h2,h2c` | A.2 |
| 3 | `< Alt-Svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000` | A.4 |

*Пояснення до цих рядків не потрібне.*

---

## Висновки

*150–300 слів. Спиратися на власні спостереження, а не на матеріал лекції.*

**D.1. Що виявилося неочевидним або несподіваним**

*Назвати конкретно, з посиланням на рядок виводу.*

> Я думав, що коли в DNS є IPv6-адреса, curl нею скористається. Але `Resolve-DnsName` показав для example.com адреси `2606:4700:...`, а в A.1 curl написав `* IPv6: (none)` і підключився по IPv4 (`Trying 172.66.147.243:443...`). Ще одне, чого я не чекав: `cf-cache-status: HIT` і `Age: 12959`. Сторінку віддав кеш Cloudflare, а не сам сервер, і вона там зберігалася близько 3,6 години.

**D.2. Чому саме така кількість груп у частині B**

*На якій підставі ухвалено рішення. Що змусило б його змінити.*

> Я зробив 5 груп за змістом рядків: вміст сторінки, запит і відповідь HTTP, захищений канал, з'єднання з адресою й портом, пошук адреси за іменем. Групу про захищений канал виділив окремо, бо в A.2 (`http://neverssl.com`) немає рядків ALPN і schannel, а решта груп є. Змінити це рішення мене змусив би рядок `schannel: remote party requests renegotiation`, який іде після `Request completely sent off`: виходить, що TLS-обмін може відбуватися вже після запиту, і групи не завжди йдуть по порядку.

**D.3. Питання, яке залишилося без відповіді**

> Я не зрозумів, яке TTL було в запису спочатку. В A.3 видно тільки залишок у кеші: 252 о 18:29 і 217 о 18:35. За 360 с воно впало лише на 35, тобто запис оновили, але з якого числа почався відлік, з мого виводу не видно.

---

## Використання штучного інтелекту

*Розділ обов'язковий. Заповнюється незалежно від того, чи використовувався ШІ. Детальні вимоги — у документі «Політика використання технологій штучного інтелекту».*

**Факт використання:** використано

**Установлений рівень для цієї роботи:** Р3 — ШІ як співвиконавець

**Фактичний рівень використання:** Р3

### Використані системи

| Система | Версія або модель | Період використання |
|---|---|---|
| Claude  | Claude Opus 5.5 | 24.11.2026 |

### Промпти

*Наводити дослівно, у тому вигляді, у якому запит було надано системі. Переказ не приймається.*

Примітка до таблиці. Промпти наведено дослівно. Повідомлення, які містили лише вставлений вивід терміналу або скріншот, позначено як «[вивід терміналу …]» / «[скріншот …]»: самі виводи повністю наведено у звіті. Повідомлення з текстами моїх відповідей позначено як «[мій текст …]»: ці тексти наведено у звіті дослівно, з виправленнями ШІ, вказаними в останній колонці.

| № | Текст промпта |
|---|---|
| 1 | Розпиши простіше інстркуцію для виконання практичної роботи з детальними поясненнями та уточненнями |
| 2 | [скріншот вікна PowerShell з помилкою запуску скрипта] |

### Дії з отриманим результатом


| № промпта | Що зроблено / що перевірено, змінено або відхилено |
|---|---|
| 1 | Спрощена версія інструкції |
| 2 | ШІ пояснив причину  |

### Підтвердження

Підтверджую, що всі наведені в цьому звіті виводи команд отримано мною особисто внаслідок фактичного виконання відповідних дій, а відомості цього розділу є повними та достовірними.

> Виводи `curl`, `dig` та інші артефакти не можуть бути згенеровані. Це стосується будь-якого рівня використання ШІ.

---

## Примітки виконавця

*(необов'язковий розділ: що не спрацювало, які команди довелося змінити, які виникли труднощі)*

> 
