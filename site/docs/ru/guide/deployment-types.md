---
next: false
---

# Long Polling против Вебхуков

Существует два способа, с помощью которых ваш бот может получать сообщения с серверов Telegram.
Они называются _long polling_ и _вебхуки_.
grammY поддерживает оба этих способа, при этом long polling используется по умолчанию.

В этом разделе мы расскажем о том, что такое long polling и вебхуки, а также о преимуществах и недостатках использования того или иного способа развертывания.
Также будет рассказано о том, как использовать их в grammY.

## Введение

Вы можете рассматривать всю дискуссию о вебхуках против long polling как вопрос о том, какой _тип развертывания_ использовать.
Другими словами, есть два принципиально разных способа разместить бота (запустить его на каком-то сервере), и они отличаются тем, как сообщения доходят до бота и могут быть обработаны grammY.

Этот выбор имеет большое значение, когда вам нужно решить, где разместить своего бота.
Например, некоторые провайдеры инфраструктуры поддерживают только один из двух типов развертывания.

Ваш бот может либо получать их (long polling), либо сервера Telegram могут передавать их вашему боту (вебхуки).

> Если вы уже знаете, как это работает, прокрутите страницу вниз, чтобы узнать, как использовать [long polling](#как-использовать-long-polling) или [вебхуки](#как-использовать-вебхуки) с помощью grammY.

## Как работает long polling?

_Представьте, что вы покупаете шарик мороженого в своем любимом магазине.
Вы подходите к сотруднику и спрашиваете свой любимый сорт мороженого.
К сожалению, он сообщает вам, что его нет в наличии._

_На следующий день вам снова захотелось вкусного мороженого, и вы снова идете в то же место и просите то же самое мороженое.
Хорошие новости!
За ночь они пополнили запасы, и вы можете насладиться мороженым с соленой карамелью уже сегодня!
Вкуснятина._

**Polling** означает, что grammY проактивно отправляет запрос в Telegram, запрашивая новые обновления (считайте: сообщения).
Если сообщений нет, Telegram вернет пустой список, указывающий на то, что с момента последнего запроса вашему боту не было отправлено ни одного нового сообщения.

Когда grammY отправляет запрос в Telegram и за это время вашему боту были отправлены новые сообщения, Telegram вернет их в виде массива, включающего до 100 объектов update.

```asciiart:no-line-numbers
______________                                   _____________
|            |                                   |           |
|            |   <--- Появились сообщения? ---   |           |
|            |    ---       не-а.         --->   |           |
|            |                                   |           |
|            |   <--- Появились сообщения? ---   |           |
|  Telegram  |    ---       не-а.         --->   |    Бот    |
|            |                                   |           |
|            |   <--- Появились сообщения? ---   |           |
|            |    ---  да, вот, держите   --->   |           |
|            |                                   |           |
|____________|                                   |___________|
```

Сразу видно, что это имеет ряд недостатков.
Ваш бот получает новые сообщения только при каждом запросе, то есть каждые несколько секунд или около того.
Чтобы бот отвечал быстрее, можно просто посылать больше запросов и не ждать так долго между ними.
Например, мы можем запрашивать новые сообщения каждую миллисекунду! Что может пойти не так...

Вместо того, чтобы решать спамить серверы Telegram, мы будем использовать _long polling_ вместо обычного (короткого) polling.

**Long polling** означает, что grammY проактивно отправляет запрос в Telegram, запрашивая новые обновления.
Если сообщений нет, Telegram будет держать соединение открытым до тех пор, пока не появятся новые сообщения, а затем ответит на запрос этими новыми сообщениями.

_Снова пора за мороженым!
Сотрудник уже приветствует вас по имени.
На вопрос о мороженом вашего любимого сорта сотрудник улыбается вам и замирает.
На самом деле, вы не получаете никакого ответа.
Поэтому вы решаете подождать, твердо улыбаясь в ответ.
И вы ждете.
И ждете._

_За несколько часов до следующего восхода солнца приезжает грузовик местной компании по доставке продуктов и заносит в подсобное помещение магазина несколько больших коробок.
Снаружи на них написано **мороженое**.
Наконец-то работник снова начинает двигаться.
"Конечно, у нас есть соленая карамель!
Две ложечки с посыпкой, как обычно?"_

_Как ни в чем не бывало, вы наслаждаетесь мороженым, покидая самый нереальный в мире магазин._

```asciiart:no-line-numbers
______________                                   _____________
|            |                                   |           |
|            |   <--- Появились сообщения? ---   |           |
|            |   .                               |           |
|            |   .                               |           |
|            |   .    *Оба терпеливо ждут*       |           |
|  Telegram  |   .                               |    Бот    |
|            |   .                               |           |
|            |   .                               |           |
|            |    ---  да, вот, держите    --->  |           |
|            |                                   |           |
|____________|                                   |___________|
```

> Обратите внимание, что в реальности ни одно соединение не будет оставаться открытым часами.
> Длительные опросные запросы имеют тайм-аут по умолчанию 30 секунд (чтобы избежать ряда [технических проблем](https://datatracker.ietf.org/doc/html/rfc6202#section-5.5)).
> Если по истечении этого времени не будет получено новых сообщений, то запрос будет отменен и отправлен заново - но общая концепция остается прежней.

Используя long polling, вам не нужно спамить сервера Telegram, и при этом вы сразу же получаете новые сообщения!
Замечательно.
Это то, что grammY делает по умолчанию, когда вы запускаете `bot.start()`.

## Как работают вебхуки?

_После этого ужасающего опыта (целая ночь без мороженого!) вы предпочитаете больше никого не спрашивать о мороженом.
Разве не было бы здорово, если бы мороженое само приходило к вам?_

Настройка **вебхука** означает, что вы предоставите Telegram URL-адрес, доступный из публичного интернета.
Всякий раз, когда вашему боту будет отправлено новое сообщение, Telegram (а не вы!) возьмет на себя инициативу и отправит запрос с объектом обновления на ваш сервер.
Мило, да?

_Вы решаете в последний раз сходить в магазин.
Вы говорите своему другу за прилавком, где вы живете.
Он обещает лично приходить к вам в квартиру, когда там появится новое мороженое (потому что оно растает на почте).
Классный парень._

```asciiart:no-line-numbers
______________                                   _____________
|            |                                   |           |
|            |                                   |           |
|            |                                   |           |
|            |        *оба терпеливо ждут*       |           |
|            |                                   |           |
|  Telegram  |                                   |    Бот    |
|            |                                   |           |
|            |                                   |           |
|            |  --- Привет, новое сообщение ---> |           |
|            |  <---    спасибо, чувак      ---  |           |
|____________|                                   |___________|
```

## Сравнение

**Основное преимущество long polling перед вебхуками заключается в том, что он проще.**
Вам не нужен домен или публичный URL.
Вам не нужно возиться с настройкой SSL-сертификатов, если вы запускаете бота на VPS.
Используйте `bot.start()`, и все будет работать, не требуя дополнительной настройки.
При нагрузке вы полностью контролируете количество обрабатываемых сообщений.

Места, где long polling работает хорошо:

- Во время разработки локально.
- На большинстве серверов.
- На хостируемых "внутренних" экземплярах, т.е. машинах, на которых ваш бот активно работает 24 часа в сутки 7 дней в неделю.

**Основное преимущество вебхуков перед long polling заключается в том, что они дешевле.**
Вы экономите тонну лишних запросов.
Вам не нужно постоянно держать открытым сетевое соединение.
Вы можете использовать сервисы, которые автоматически сводят инфраструктуру к нулю при отсутствии запросов.
При желании можно даже [сделать вызов API при ответе на запрос Telegram](#ответ-вебхука), хотя это и имеет ряд недостатков.
Ознакомьтесь с настройкой [здесь](/ref/core/apiclientoptions#canusewebhookreply).

Места, где вебхуки работают хорошо:

- На серверах с SSL-сертификатами.
- На размещенных "внешних" экземплярах, которые масштабируются в зависимости от нагрузки.
- На бессерверных платформах, таких как облачные функции или программируемые пограничные сети.

## Я до сих пор не знаю, что использовать

Тогда выбирайте long polling.
Если у вас нет веских причин использовать вебхуки, обратите внимание, что у long polling нет серьезных недостатков, и, согласно нашему опыту, вы потратите гораздо меньше времени на исправление ошибок.
Время от времени вебхуки могут быть немного неприятными (см. [ниже](#своевременное-завершение-запросов-вебхуков)).

Что бы вы ни выбрали, если у вас возникнут серьезные проблемы, переключиться на другой тип развертывания будет несложно.
В случае с grammY вам придется изменить всего несколько строк кода.
Настройка вашего [middleware](./middleware) не изменится.

## Как использовать long polling

Вызовите

```ts
bot.start();
```

для запуска вашего бота с очень простой формой long polling.
Он обрабатывает все обновления последовательно.
Это делает вашего бота очень простым в отладке, а его поведение - очень предсказуемым, поскольку в нем нет параллелизма.

Если вы хотите, чтобы ваши сообщения обрабатывались grammY одновременно, или вас беспокоит пропускная способность, ознакомьтесь с разделом о [grammY runner](../plugins/runner).

## Как использовать вебхуки

Если вы хотите запустить grammY с помощью вебхуков, вы можете интегрировать бота в веб-сервер.
Поэтому мы ожидаем, что вы сможете запустить простой веб-сервер с выбранным вами фреймворком.

Каждый бот grammY может быть преобразован в middleware для ряда веб-фреймворков, включая `express`, `koa`/`oak` и другие.
Вы можете импортировать функцию `webhookCallback` ([документация API](/ref/core/webhookcallback)), чтобы создать middleware для соответствующего фреймворка.

::: code-group

```ts [TypeScript]
import express from "express";

const app = express(); // или то, что вы используете
app.use(express.json()); // спарсите тело JSON запроса

// "express" также используется по умолчанию, если аргумент не указан.
app.use(webhookCallback(bot, "express"));
```

```js [JavaScript]
const express = require("express");

const app = express(); // или то, что вы используете
app.use(express.json()); // спарсите тело JSON запроса

// "express" также используется по умолчанию, если аргумент не указан.
app.use(webhookCallback(bot, "express"));
```

```ts [Deno]
import { Application } from "https://deno.land/x/oak/mod.ts";

const app = new Application(); // или то, что вы используете

// Обязательно укажите, какой фреймворк вы используете.
app.use(webhookCallback(bot, "oak"));
```

:::

> Обратите внимание, что вы не должны вызывать `bot.start()` при использовании webhooks.

Теперь ваше приложение прослушивает запросы вебхуков от Telegram.
Последнее, что вам нужно сделать, это указать Telegram, куда отправлять обновления.
Есть несколько способов сделать это, но в конечном итоге все они просто вызывают `setWebhook`, как описано [здесь](https://core.telegram.org/bots/api#setwebhook).

Самый простой способ установить вебхук --- вставить следующий URL в адресную строку браузера, заменив `<токен>` на токен вашего бота, а `<url>` на публичную конечную точку вашего сервера.

```txt
https://api.telegram.org/bot<токен>/setWebhook?url=<url>
```

Мы также создали соответствующий интерфейс для этого, если вы предпочитаете управлять вебхуком через веб-сайт.
Вы можете найти его здесь: <https://telegram.tools/webhook-manager>

Обратите внимание, что вы также можете установить свой вебхук из кода:

```ts
const endpoint = ""; // <-- поместите сюда свой URL
await bot.api.setWebhook(endpoint);
```

Наконец, обязательно прочитайте [замечательное руководство Марвина по всем вещам, связанным с вебхуками](https://core.telegram.org/bots/webhooks), написанное командой Telegram, если вы рассматриваете [запуск бота на вебхуках на VPS](../hosting/vps#запуск-бота-на-вебхуках).

### Адаптеры для веб-фреймворков

Для того чтобы поддерживать множество различных веб-фреймворков, в grammY используется концепция **адаптеров**.
Каждый адаптер отвечает за передачу входных и выходных данных от веб-фреймворка к grammY и наоборот.
Второй параметр, передаваемый в `webhookCallback` ([документация API](/ref/core/webhookcallback)), определяет адаптер фреймворка, используемый для связи с веб-фреймворком.

Из-за того, что этот подход работает, нам обычно нужен адаптер для каждого фреймворка, но, поскольку некоторые фреймворки имеют схожий интерфейс, существуют адаптеры, которые, как известно, работают с несколькими фреймворками.
Ниже приведена таблица с доступными на данный момент адаптерами, а также фреймворками, API или режимами выполнения, с которыми они работают.

| Адаптер            | Фреймворк/API/Среда выполнения                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| `aws-lambda`       | AWS Lambda Functions                                                                                |
| `aws-lambda-async` | AWS Lambda Functions с `async`/`await`                                                              |
| `azure`            | Azure Functions                                                                                     |
| `bun`              | `Bun.serve`                                                                                         |
| `cloudflare`       | Cloudflare Workers                                                                                  |
| `cloudflare-mod`   | Cloudflare Module Workers                                                                           |
| `express`          | Express, Google Cloud Functions                                                                     |
| `fastify`          | Fastify                                                                                             |
| `hono`             | Hono                                                                                                |
| `http`, `https`    | Node.js `http`/`https` modules, Vercel Serverless                                                   |
| `koa`              | Koa                                                                                                 |
| `next-js`          | Next.js                                                                                             |
| `nhttp`            | NHttp                                                                                               |
| `oak`              | Oak                                                                                                 |
| `serveHttp`        | `Deno.serveHttp`                                                                                    |
| `std/http`         | `Deno.serve`, `std/http`, `Deno.upgradeHttp`, `Fresh`, `Ultra`, `Rutt`, `Sift`, Vercel Edge Runtime |
| `sveltekit`        | SvelteKit                                                                                           |
| `worktop`          | Worktop                                                                                             |

### Ответ вебхука

При получении запроса на вебхук ваш бот может вызвать до одного метода в ответе.
Преимуществом является то, что это избавляет вашего бота от необходимости делать до одного HTTP-запроса на каждое обновление.
Однако у этого способа есть ряд недостатков:

1. Вы не сможете обработать возможные ошибки соответствующего вызова API.
   К ним относятся ошибки ограничения скорости, поэтому вы не сможете гарантировать, что ваш запрос будет иметь какой-либо эффект.
2. Что еще более важно, у вас также не будет доступа к объекту ответа.
   Например, вызов `sendMessage не даст вам доступа к отправленному сообщению.
3. Кроме того, невозможно отменить запрос.
   Сигнал `AbortSignal` будет проигнорирован.
4. Обратите внимание, что типы в grammY не отражают последствий выполненного обратного вызова вебхука!
   Например, они указывают на то, что вы всегда получаете объект ответа, так что вы сами должны убедиться в том, что вы не облажаетесь при использовании этой незначительной оптимизации производительности.

Если вы хотите использовать вебхук-ответы, вы можете указать опцию `canUseWebhookReply` в опции `client` вашего `BotConfig` ([документация API](/ref/core/botconfig)).
Передайте функцию, которая определяет, использовать или нет ответ вебхука для данного запроса, идентифицированного методом.

```ts
const bot = new Bot("", {
  client: {
    // Мы принимаем недостаток ответов веб-хуков для ввода статуса.
    canUseWebhookReply: (method) => method === "sendChatAction",
  },
});
```

Вот как работают ответы на вебхуки под капотом.

```asciiart:no-line-numbers
______________                                   _____________
|            |                                   |           |
|            |                                   |           |
|            |                                   |           |
|            |       *оба терпиливо ждут*        |           |
|            |                                   |           |
|  Telegram  |                                   |    Бот    |
|            |                                   |           |
|            |                                   |           |
|            |   --- новое сообщение      --->   |           |
|            |  <--- окей, sendChatAction ---    |           |
|____________|                                   |___________|
```

### Своевременное завершение запросов вебхуков

> Вы можете игнорировать остальную часть этой страницы, если все ваши middleware завершаются быстро, т.е. в течение нескольких секунд.
> Этот раздел предназначен в первую очередь для тех, кто хочет выполнять передачу файлов в ответ на сообщения или другие операции, требующие больше времени.

Когда Telegram отправляет обновление из одного чата вашему боту, он будет ждать, пока вы завершите запрос, прежде чем доставить следующее обновление, относящееся к этому чату.
Другими словами, Telegram будет доставлять обновления из одного и того же чата последовательно, а обновления из разных чатов отправляются параллельно.
(Источник этой информации --- [здесь](https://github.com/tdlib/telegram-bot-api/issues/75#issuecomment-755436496)).

Telegram старается сделать так, чтобы ваш бот получал все обновления.
Это означает, что если доставка обновлений в чат не удалась, последующие обновления будут стоять в очереди до тех пор, пока не удастся получить первое обновление.

#### Почему не завершать запрос вебхука опасно

У Telegram есть тайм-аут для каждого обновления, которое он отправляет на конечную точку вебхука.
Если вы не завершите запрос вебхука достаточно быстро, Telegram повторно отправит обновление, считая, что оно не было доставлено.
В результате ваш бот может неожиданно обработать одно и то же обновление несколько раз.
Это означает, что он будет выполнять всю обработку обновления, включая отправку любых ответных сообщений, несколько раз.

```asciiart:no-line-numbers
______________                                   _____________
|            |                                   |           |
|            | ---    новое сообщение    --->    |           |
|            |                              .    |           |
|            |        *бот обрабатывает*    .    |           |
|            |                              .    |           |
|  Telegram  | --- Я сказал сообщение!!! --->    |    Бот    |
|            |                              ..   |           |
|            |    *бот обрабатывает дважды* ..   |           |
|            |                              ..   |           |
|            | ---      АЛЛЛЛОООООО      --->    |           |
|            |                              ...  |           |
|            |   *бот обрабатывает трижды*  ...  |           |
|____________|                              ...  |___________|
```

Именно поэтому в функции `webhookCallback` у grammY есть свой собственный, более короткий тайм-аут (по умолчанию: 10 секунд).
Если ваш middleware завершит работу раньше, функция `webhookCallback` ответит на вебхук автоматически.
В этом случае все в порядке.
Однако если ваш middleware не завершится до истечения тайм-аута, установленного grammY, `webhookCallback` выбросит ошибку.
Это означает, что вы можете обработать ошибку в своем веб-фреймворке.
Если у вас нет такой обработки ошибок, Telegram снова отправит то же самое обновление - но, по крайней мере, у вас теперь будут журналы ошибок, чтобы сообщить вам, что что-то не так.

Когда Telegram отправит обновление вашему боту во второй раз, маловероятно, что ваша обработка будет быстрее, чем в первый раз.
В результате, скорее всего, снова произойдет тайм-аут, и Telegram снова отправит обновление.
Таким образом, ваш бот увидит обновление не просто два раза, а несколько десятков раз, пока Telegram не прекратит повторные попытки.
Вы можете заметить, что ваш бот начнет спамить пользователей, поскольку он пытается обработать все эти обновления (которые на самом деле каждый раз одни и те же).

#### Почему досрочное завершение запроса вебхука также опасно

Вы можете настроить `webhookCallback` так, чтобы он не выбрасывал ошибку по истечении тайм-аута, а завершал запрос вебхука досрочно, даже если ваш middleware все еще работает.
Это можно сделать, передав `"return"` в качестве третьего аргумента в `webhookCallback`, вместо значения по умолчанию `"throw"`.
Однако, несмотря на то, что такое поведение имеет несколько правильных вариантов использования, подобное решение обычно вызывает больше проблем, чем решает.

Помните, что как только вы ответите на запрос вебхука, Telegram отправит следующее обновление для этого чата.
Однако, поскольку старое обновление все еще обрабатывается, два обновления, которые ранее обрабатывались последовательно, внезапно обрабатываются параллельно.
Это может привести к возникновению условий гонки.
Например, плагин сессии неизбежно сломается из-за опасности [WAR](https://en.wikipedia.org/wiki/Hazard_(computer_architecture)#Write_after_read_(WAR)).
**Это приводит к потере данных!**
Другие плагины и даже ваш собственный middleware тоже может сломаться.
Степень этого неизвестна и зависит от вашего бота.

#### Как решить эту проблему

Этот ответ легче сказать, чем сделать.
**Ваша работа заключается в том, чтобы убедиться, что ваш middleware завершается достаточно быстро.**
Не используйте долго выполняющиеся middleware.
Да, мы знаем, что вы, возможно, _хотите_ иметь долго выполняющиеся задачи.
Но все же.
Не делайте этого.
Только не в вашем middleware.

Вместо этого используйте очередь (существует множество систем очередей, от очень простых до очень сложных).
Вместо того чтобы пытаться выполнить всю работу за небольшое окно тайм-аута вебхука, просто добавьте задачу в очередь для отдельной обработки и позвольте вашему middleware завершить работу.
Очередь может использовать столько времени, сколько захочет.
Когда она закончит, она может отправить сообщение обратно в чат.
Это несложно сделать, если вы используете простую очередь в памяти.
Это может быть немного сложнее, если вы используете отказоустойчивую внешнюю систему очередей, которая сохраняет состояние всех задач и может повторить выполнение, даже если ваш сервер внезапно умрет.

```asciiart:no-line-numbers
______________                                   _____________
|            |                                   |           |
|            |   ---    новое сообщение   --->   |           |
|            |  <---    спасибо, чувак    ---.   |           |
|            |                               .   |           |
|            |                               .   |           |
|  Telegram  |      *очередь бота работает*  .   |    Бот    |
|            |                               .   |           |
|            |                               .   |           |
|            |  <---  результат очереди   ---    |           |
|            |   ---     отличненько      --->   |           |
|____________|                                   |___________|
```

#### Почему `"return"` в целом хуже, чем `"throw"`

Вам может быть интересно, почему по умолчанию `webhookCallback` выбрасывает ошибку, а не завершает запрос успешно.
Такой выбор был сделан по следующим причинам.

Условия гонки очень трудно воспроизвести, и они могут возникать крайне редко или периодически.
Решение этой проблемы заключается в том, чтобы _убедиться, что тайм-ауты не возникают_ в первую очередь.
Но если вы столкнетесь с ними, вы должны знать, что это происходит, чтобы вы могли исследовать и устранить проблему!
По этой причине вы хотите, чтобы ошибка появлялась в ваших логах.
Установка обработчика тайм-аута в `"return"`, то есть подавление тайм-аута и притворство, что ничего не произошло, прямо противоположно полезному поведению.

Если вы сделаете это, то в некотором смысле будете использовать очередь обновлений в вебхуке доставки Telegram в качестве очереди задач.
Это плохая идея по всем причинам, описанным выше.
То, что grammY _может_ упускать ошибки, из-за которых вы можете потерять данные, не означает, что вы должны _указывать_ ему это делать.
Этот параметр конфигурации не следует использовать в случаях, когда вашему middleware просто требуется слишком много времени для завершения работы.
Потратьте время на то, чтобы правильно исправить эту проблему, и ваши будущие я (и пользователи) будут вам благодарны.
