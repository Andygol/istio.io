---
title: HBONE
description: Розуміння протоколу безпечного тунелювання Istio.
weight: 4
owner: istio/wg-networking-maintainers
test: no
---

**HBONE** (HTTP-Based Overlay Network Environment) — це безпечний тунельний протокол, що використовується між компонентами Istio. HBONE є специфічним терміном Istio. Це механізм для прозорого мультиплексування TCP-потоків, повʼязаних з багатьма різними застосунками, через одне зашифроване зʼєднання: зашифрований тунель.

У своїй поточній реалізації в Istio протокол HBONE поєднує три відкриті стандарти:

- [HTTP/2](https://httpwg.org/specs/rfc7540.html)
- [HTTP CONNECT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/CONNECT)
- [Mutual TLS (mTLS)](https://datatracker.ietf.org/doc/html/rfc8446)

HTTP CONNECT використовується для встановлення тунельного зʼєднання, mTLS — для забезпечення та шифрування цього зʼєднання, а HTTP/2 — для мультиплексування потоків застосунків через це одне захищене та зашифроване зʼєднання і передачі додаткових метаданих на рівні потоків.

## Безпека та оренда {#security-and-tenancy}

Як зазначено у специфікації mTLS, кожне тунельне зʼєднання повинно мати унікальні ідентифікатори джерела та призначення, і ці ідентифікатори повинні використовуватися для встановлення шифрування для цього зʼєднання.

Це означає, що зʼєднання застосунків через протокол HBONE до одного й того ж ідентифікатора призначення будуть мультиплексовані через одне спільне, зашифроване та захищене зʼєднання HTTP/2. Фактично, кожне унікальне джерело та призначення повинні отримати своє власне захищене тунельне зʼєднання, навіть якщо це зʼєднання, що його забезпечує, обробляє кілька зʼєднань на рівні застосунків.

## Деталі реалізації {#implementation-details}

Згідно з конвенцією Istio, ztunnel та інші проксі, що розуміють протокол HBONE, експонують слухачів на TCP-порті 15008.

Оскільки HBONE є комбінацією HTTP/2, HTTP CONNECT та mTLS, пакети тунелів HBONE, що проходять між проксі, що підтримують HBONE, виглядають наступним чином:

{{< image width="100%"
link="hbone-packet.svg"
caption="Формат пакету HBONE L3"
>}}

Важливою властивістю тунелю HBONE є те, що початковий запит застосунку може бути прозоро проксійований без зміни основного потоку трафіку застосунку. Це означає, що метадані про зʼєднання можуть передаватися до призначених проксі без зміни запиту застосунку — наприклад, без необхідності додавати специфічні для Istio заголовки до трафіку застосунку.

Додаткові варіанти використання HBONE та HTTP-тунелювання (такі як UDP) будуть досліджені в майбутньому, оскільки режим ambient та стандарти еволюціонують.
