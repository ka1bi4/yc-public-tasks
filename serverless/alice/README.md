## Yandex Cloud Functions

### Создайте функцию

Для того, чтобы создать функцию, перейдите в
[консоль Яндекс.Облака](https://console.cloud.yandex.ru/),
зайдите в раздел Cloud Functions и нажмите "Создать функцию".
 
Введите уникальное имя для функции, к примеру,
`my-alice-skill-MYNAME` и нажмите "Создать".

### Сделайте функцию публичной

Для этого зайдите на страницу Вашей функции и нажмите на переключатель
"Публичная функция". Таким образом Вашу функцию можно будет вызывать
без авторизации.

### Создайте наполнение функции

Чтобы создать новую версию функции и задать параметры выполнения и код,
нажмите на кнопку "Создать в редакторе" (или перейдите в раздел "Редактор")
в левой панели навигации.

Добавьте новый файл с именем `alice.js`. Для этого нажмите "Создать файл"
и во всплывающем окне введите имя файла `alice.js`. Когда файл будет создан,
он автоматически откроется в редакторе.

Вставьте код обработчика в область редактирования:

```
const {TranslationService, TranslateRequest} = require('yandex-cloud/api/ai/translate/v2');

const translationService = new TranslationService();

function sendResponse(event, text, endSession) {
    const {version, session, request} = event;
    return {
        version,
        session,
        response: {
            text: text,
            end_session: endSession === true,
        },
    }
}

async function translate(text) {
    let response = await translationService.translate({
        targetLanguageCode: 'en',
        format: TranslateRequest.Format.PLAIN_TEXT,
        texts: [text],
    });
    return response.translations[0].text;
}

async function handler(event) {
    let query = event.request['original_utterance'];
    if (!query || query.length === 0) {
        return sendResponse(event, 'Привет! Я переведу на анлийский всё, что вы мне cкажете.');
    }
    query = query.replace(/^переведи\s?/, '');

    const translated = await translate(query);
    return sendResponse(
        event,
        `${query} по-английски будет ${translated}. Обращайтесь ещё!`,
        true,
    );
}

module.exports = {
    handler,
};
```

Далее необходимо задать параметры необходимые для исполнения функции:

* Точка входа: `alice.handler` (имя файла без расширения и имя Вашей функции);
* Сервисный аккаунт: выберите из выпадающего списка предсозданный аккаунт (или создайте новый).

Все остальные параметры можно оставить со значениями по-умолчанию, эти параметры
идеально подходят для навыка Алисы.

Сохраните изменения в виде новой версии, для этого наверху страницы нажмите "Создать версию".

### Создайте навык для Алисы

Зайдите в консоль [управления навыками Яндекс.Диалогов](https://dialogs.yandex.ru/developer)
и нажмите "Создать диалог", в выпадающем списке укажите "Навык Алисы".

Придумайте имя для Вашего навыка. К примеру, "Навык ВАШЕ ИМЯ".

В разделе "Примеры запросов" укажите "Запусти навык" и выберите имя вашего навыка. В разделе
"Backend" выберите опцию "Функция в Яндекс.Облаке" и в выпадающем списке выберите свою функцию.

Заполните необходимую информацию в разделе "Публикация в каталоге". Если вы отметите "Не публиковать в каталоге",
то можете не указывать реальные данные. Пример иконки для навыка Вы можете найти на рабочем столе. 

Нажмите "Сохранить".

### Проверьте работу своего навыка

Перейдите на вкладку "Тестирование" и попробуйте чат с Алисой,
Ваш навык будет переводить на английский все что Вы напишете в чат.