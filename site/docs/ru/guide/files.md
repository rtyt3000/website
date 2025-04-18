# Обработка файлов

Боты Telegram могут не только отправлять и получать текстовые сообщения, но и многие другие виды сообщений, например, фото и видео.
Это предполагает работу с файлами, которые прикрепляются к сообщениям.

## Как работают файлы для ботов Telegram

> Этот раздел объясняет, как работают файлы для ботов Telegram.
> Если вы хотите узнать, как можно работать с файлами в grammY, прокрутите страницу вниз, чтобы узнать о [скачивании](#получение-фаилов) и [загрузке](#отправка-фаилов) файлов.

Файлы хранятся отдельно от сообщений.
Файл на серверах Telegram идентифицируется по `file_id`, который представляет собой длинную строку символов.
Например, она может выглядеть как `AgADBAADZRAxGyhM3FKSE4qKa-RODckQHxsoABDHe0BDC1GzpGACAAEC`.

### Идентификаторы для полученных файлов

> Боты получают только идентификаторы файлов.
> Если они хотят получить содержимое файла, они должны запросить его напрямую.

Когда ваш бот **получает** сообщение с файлом, на самом деле он получает не все данные файла, а только `file_id`.
Если ваш бот действительно хочет загрузить файл, то он может сделать это, вызвав метод `getFile` ([документация Telegram Bot API](https://core.telegram.org/bots/api#getfile)).
Этот метод позволяет загрузить файл, сконструировав специальный временный URL.
Обратите внимание, что этот URL гарантированно будет действителен только в течение 60 минут, после чего его срок действия может истечь. В этом случае вы можете просто вызвать `getFile` снова.

Файлы могут быть получены вот [так](#получение-фаилов).

### Идентификаторы для отправленных файлов

> Отправляя файлы, вы также получаете идентификатор файла.

Всякий раз, когда ваш бот **отправляет** сообщение с файлом, он получает информацию об отправленном сообщении, включая `file_id` отправленного файла.
Это означает, что все файлы, которые видит бот, как при отправке, так и при получении, будут иметь `file_id`.
Если вы хотите работать с файлом после того, как ваш бот его увидит, вы всегда должны хранить его `file_id`.

> Используйте идентификаторы файлов всегда, когда это возможно.
> Они очень эффективны.

Когда бот отправляет сообщение, он может **указать `file_id`, который он уже видел**.
Это позволит ему отправить идентифицированный файл, без необходимости загружать данные для него.

Вы можете использовать один и тот же `file_id` сколько угодно раз, так что вы можете отправить один и тот же файл в пять разных чатов, используя один и тот же `file_id`.
Однако вы должны убедиться, что используете правильный метод - например, вы не можете использовать `file_id`, который идентифицирует фотографию, при вызове [`sendVideo`](https://core.telegram.org/bots/api#sendvideo).

Файлы могут быть отправлены вот [так](#отправка-фаилов).

### Идентификаторы которые могут вас удивить

> Идентификаторы файлов **работают только для вашего бота**.
> Если другой бот использует ваши идентификаторы файлов, он может случайно сработать, случайно разбиться и случайно убить невинных котят.
> :cat: → :skull:

Каждый бот имеет свой собственный набор `file_id` для файлов, к которым он может получить доступ.
Вы не сможете надежно использовать `file_id` от бота вашего друга для доступа к файлу с помощью _вашего_ бота.
Каждый бот будет использовать разные идентификаторы для одного и того же файла.
Это означает, что вы не можете просто угадать `file_id` и получить доступ к файлу какого-то случайного человека, потому что Telegram отслеживает, какие `file_id` действительны для вашего бота.

::: warning Использование других `file_id`
Обратите внимание, что в некоторых случаях технически возможно, что `file_id` от другого бота работает корректно.
**Однако**, использование чужих `file_id` опасно, так как они могут перестать работать в любой момент без предупреждения.
Поэтому всегда убеждайтесь, что все `file_id`, которые вы используете, были изначально предназначены для вашего бота.
:::

> У файла может быть несколько идентификаторов.

С другой стороны, возможно, что бот в конечном итоге видит один и тот же файл, идентифицированный разными `file_id`.
Это означает, что вы не можете полагаться на сравнение `file_id` для проверки того, являются ли два файла одним и тем же.
Если вам нужно идентифицировать один и тот же файл в течение долгого времени (или для нескольких ботов), вы должны использовать значение `file_unique_id`, которое ваш бот получает вместе с каждым `file_id`.

Значение `file_unique_id` не может быть использовано для загрузки файлов, но оно будет одинаковым для любого файла у всех ботов.

## Получение файлов

Вы можете работать с файлами так же, как и с любыми другими сообщениями.
Например, если вы хотите прослушать голосовые сообщения, вы можете сделать следующее:

```ts
bot.on("message:voice", async (ctx) => {
  const voice = ctx.msg.voice;

  const duration = voice.duration; // в секундах
  await ctx.reply(`Ваше сообщение длиной ${duration} секунд(ы).`);

  const fileId = voice.file_id;
  await ctx.reply("Идентификатор файла вашего голосового сообщения: " + fileId);

  const file = await ctx.getFile(); // Действует не менее 1 часа
  const path = file.file_path; // путь к файлу на API сервере бота
  await ctx.reply("Скачайте свой собственный файл снова: " + path);
});
```

::: tip Передача пользовательского идентификатора файла в getFile
На объекте контекста `getFile` является [краткой записью](./context#краткая-запись), и будет получать информацию о файле текущего сообщения.
Если вы хотите получить другой файл при работе с сообщением, используйте `ctx.api.getFile(file_id)` вместо этого.
:::

> Посмотрите краткие записи [`:media` и `:file`](./filter-queries#сокращения) для фильтровую запросов, если вы хотите получить любой тип файла.

Вызвав `getFile`, вы можете использовать возвращаемый `file_path` для загрузки файла по этому URL `https://api.telegram.org/file/bot<токен>/<file_path>`, где `<токен>` должен быть заменен на ваш токен бота.

Если вы [запускаете свой собственный API сервер](./api#запуск-локального-api-сервера-бота), то `file_path` будет представлять собой абсолютный путь к файлу, который указывает на файл на вашем локальном диске.
В этом случае вам больше не нужно ничего скачивать, так как сервер Bot API скачает файл за вас при вызове `getFile`.

::: tip Плагин Files
grammY не поставляется в комплекте с собственным загрузчиком файлов, но вы можете установить [официальный плагин files](../plugins/files).
Это позволит вам загружать файлы через `await file.download()`, а также получать URL для их загрузки через `file.getUrl()`.
:::

## Отправка файлов

У ботов Telegram есть [три способа](https://core.telegram.org/bots/api#sending-files) отправки файлов:

1. Через `file_id`, то есть отправляя файл по идентификатору, который уже известен боту.
2. Через URL, т.е. передав публичный URL файла, который Telegram скачает и отправит за вас.
3. Через загрузку собственного файла.

Во всех случаях методы, которые вам нужно вызвать, называются одинаково.
В зависимости от того, какой из трех способов отправки файла вы выберете, параметры этих функций будут отличаться.
Например, чтобы отправить фотографию, вы можете использовать `ctx.replyWithPhoto` (или `sendPhoto`, если вы используете `ctx.api` или `bot.api`).

Вы можете отправлять файлы других типов, просто переименовав метод и изменив тип передаваемых в него данных.
Чтобы отправить видео, вы можете использовать `ctx.replyWithVideo`.
То же самое касается и документа: `ctx.replyWithDocument`.
Вы поняли суть.

Давайте разберемся, что представляют собой эти три способа отправки файла.

### Через `file_id` или URL

Первые два метода просты: вы просто передаете соответствующее значение в виде `строки`, и все готово.

```ts
// Отправка через file_id.
await ctx.replyWithPhoto(existingFileId);

// Отправка через URL.
await ctx.replyWithPhoto("https://grammy.dev/images/grammY.png");

// В качестве альтернативы можно использовать bot.api.sendPhoto() или ctx.api.sendPhoto().
```

### Загрузка ваших собственных файлов

В grammY есть хорошая поддержка загрузки собственных файлов.
Для этого нужно импортировать и использовать класс `InputFile` ([документация grammY API](/ref/core/inputfile)).

```ts
// Отправка файла по локальному пути
await ctx.replyWithPhoto(new InputFile("/tmp/picture.jpg"));

// В качестве альтернативы используйте bot.api.sendPhoto() или ctx.api.sendPhoto()
```

Конструктор `InputFile` принимает не только пути к файлам, но и потоки, объекты `Buffer`, асинхронные итераторы и, в зависимости от вашей платформы - многое другое, или функцию, которая создает любую из этих вещей.
Все, что вам нужно помнить, это: **создайте экземпляр `InputFile` и передайте его в любой метод для отправки файла**.
Экземпляры `InputFile` можно передавать всем методам, которые принимают отправку файлов путем загрузки.

Вот несколько примеров того, как можно создать `InputFile`.

#### Загрузка файлов с диска

Если у вас уже есть файл, хранящийся на вашем компьютере, вы можете позволить grammY загрузить этот файл.

::: code-group

```ts [Node.js]
import { createReadStream } from "fs";

// Отправка локального файла.
new InputFile("/path/to/file");

// Отправка потоком.
new InputFile(createReadStream("/path/to/file"));
```

```ts [Deno]
// Отправка локального файла.
new InputFile("/path/to/file");

// Отправка из экземпляра `Deno.FsFile`.
new InputFile(await Deno.open("/path/to/file"));
```

:::

#### Загрузка сырых бинарных данных

Вы также можете отправить объект `Buffer` или итератор, который выдает объекты `Buffer`.
На Deno вы также можете отправлять объекты `Blob`.

::: code-group

```ts [Node.js]
// Отправьте буфер или массив байтов.
const buffer = Uint8Array.from([65, 66, 67]);
new InputFile(buffer); // "ABC"
// Отправьте итерируемый объект.
new InputFile(function* () {
  // "ABCABCABCABC"
  for (let i = 0; i < 4; i++) yield buffer;
});
```

```ts [Deno]
// Send a blob.
const blob = new Blob(["ABC"], { type: "text/plain" });
new InputFile(blob);
// Отправьте буфер или массив байтов.
const buffer = Uint8Array.from([65, 66, 67]);
new InputFile(buffer); // "ABC"
// Отправьте итерируемый объект.
new InputFile(function* () {
  // "ABCABCABCABC"
  for (let i = 0; i < 4; i++) yield buffer;
});
```

:::

#### Загрузка и повторая отправка файлов

Вы даже можете заставить grammY загрузить файл из Интернета.
При этом файл не будет сохранен на диске.
Вместо этого grammY будет только передавать данные, сохраняя в памяти лишь небольшой фрагмент.
Это очень эффективно.

> Обратите внимание, что Telegram поддерживает загрузку файла многими способами.
> Если возможно, вы должны предпочесть [отправить файл по URL](#через-file-id-или-url), а не использовать `InputFile` для передачи содержимого файла через ваш сервер.

```ts
// Загрузите файл и передайте ответ в Telegram.
new InputFile(new URL("https://grammy.dev/images/grammY.png"));
new InputFile({ url: "https://grammy.dev/images/grammY.png" }); // равносильно
```

### Добавление подписи

При отправке файлов вы можете указать дополнительные параметры в объекте options типа `Other`, точно так же, как объяснялось [ранее](./basics#отправка-сообщении).
Например, это позволяет отправлять подписи.

```ts
// Отправьте фотографию из локального файла пользователю с ID 12345 с подписью "photo.jpg".
await bot.api.sendPhoto(12345, new InputFile("/path/to/photo.jpg"), {
  caption: "photo.jpg",
});
```

Как всегда, как и все остальные методы API, вы можете отправлять файлы через `ctx` (самый простой), `ctx.api` или `bot.api`.

## Лимиты размеров файла

Сам grammY может отправлять файлы без ограничений по размеру, однако Telegram ограничивает размер файлов, как описано [здесь](https://core.telegram.org/bots/api#sending-files).
Это означает, что ваш бот не сможет скачивать файлы размером более 20 МБ или загружать файлы размером более 50 МБ.
Некоторые комбинации имеют еще более строгие ограничения, например, фотографии, отправленные по URL (5 МБ).

Как уже упоминалось в [предыдущем разделе](./api), ваш бот может работать с большими файлами, приложив некоторые дополнительные усилия.
Если вы хотите поддерживать загрузку файлов размером до 2000 МБ (максимальный размер файла в Telegram) и скачивание файлов любого размера ([4000 МБ с Telegram Премиум](https://t.me/premium/5)), вам необходимо [разместить собственный API сервер](./api#запуск-локального-api-сервера-бота) в дополнение к хостингу вашего бота.

Хостинг собственного API сервера бота сам по себе не имеет никакого отношения к grammY.
Однако grammY поддерживает все методы, необходимые для настройки вашего бота на использование собственного API сервера.
