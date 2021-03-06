# Различайте операционные ошибки и ошибки программиста

### Объяснение в один абзац

Разделение ошибок на эти два типа минимизирует время простоя вашего приложения и помогает избежать сумасшедших ошибок: операционные ошибки относятся к ситуациям, когда вы понимаете происходящее и влияние ошибки на программу. Например, запрос к какой-нибудь службе HTTP не выполнен из-за проблемы с подключением. С другой стороны, ошибки программиста относятся к случаям, когда вы не знаете, почему, а иногда и откуда возникла ошибка - это может быть какой-то код, который пытался прочитать undefined-значение или код, который использует пул соединений с БД и приводит к утечке памяти. Операционные ошибки относительно легко обрабатываются - обычно достаточно логирования ошибки. Хуже становиться, когда появляется ошибка программиста, приложение может быть в несогласованном состоянии, и все, что вы можете сделать - перезапустить приложение.

### Пример кода - помечаем ошибку как операционную (доверенную)

<details>
<summary><strong>Javascript</strong></summary>

```javascript
// помечаем объект ошибки, как операционный
const myError = new Error('Как мы можем добавить продукт, если значение не задано?');
myError.isOperational = true;

// или, если вы используете централизированную фабрику ошибок (смотрите другие примеры в статье "Используйте только встроенный объект Error")
class AppError {
  constructor (commonType, description, isOperational) {
    Error.call(this);
    Error.captureStackTrace(this);
    this.commonType = commonType;
    this.description = description;
    this.isOperational = isOperational;
  }
};

throw new AppError(errorManagement.commonErrors.InvalidInput, 'Описываем, что произошло', true);

```
</details>

<details>
<summary><strong>Typescript</strong></summary>

```typescript
// или, если вы используете централизированную фабрику ошибок (смотрите другие примеры в статье "Используйте только встроенный объект Error")
export class AppError extends Error {
  public readonly commonType: string;
  public readonly isOperational: boolean;

  constructor(commonType: string, description: string, isOperational: boolean) {
    super(description);

    Object.setPrototypeOf(this, new.target.prototype); // восстанавливаем цепочку прототипов

    this.commonType = commonType;
    this.isOperational = isOperational;

    Error.captureStackTrace(this);
  }
}

// помечаем объект ошибки, как операционный (true)
throw new AppError(errorManagement.commonErrors.InvalidInput, 'Describe here what happened', true);

```
</details>

### Цитата блога: "Ошибки программиста - это ошибки в программе"

Из блога Джойент, занявшего 1 место по ключевым словам "Обработка ошибок Node.js"

> … Лучший способ исправить ошибки программиста - это сразу же аварийно завершить работу. Вы должны запускать свои программы, используя сервис перезапуска, который автоматически будет перезапускать программу в случае сбоя. Сервис перезапуска является самым простым способом надежного восстановления сервиса при возникновении временной ошибки программиста…

### Цитата из блога: "Нет безопасного способа продолжить без возникновения хрупкого undefined-состояния системы"

Из официальной документации Node.js

> …По самой природе того, как throw работает в JavaScript, почти всегда нет способа безопасно "начать оттуда, где мы остановились", без утечки ссылок или создания другого undefined-состояния. Самый безопасный способ отреагировать на возникшую ошибку - завершить процесс. Конечно, на обычном веб-сервере у вас может быть открыто много подключений, и нет смысла внезапно закрывать все, потому что кто-то другой выбросил ошибку. Лучший вариант - отправить ответ об ошибке на запрос, который ее вызвал, позволяя остальным завершить работу в удобное время, и после прекратить прослушивание новых запросов в воркере, в котором возникла ошибка.

### Цитата блога: "В противном случае вы рискуете состоянием вашего приложения"

Из блога debugable.com, занявшего 3 место по ключевым словам "Node.js uncaught exception"

> …Итак, если вы действительно не знаете, что делаете, вам следует выполнить постепенный перезапуск службы после получения события исключения "uncaughtException". В противном случае вы рискуете привести к несогласованности состояния вашего приложения или сторонних библиотек, что приведет к всевозможным сумасшедшим ошибкам…

### Цитата блога: "Есть три школы мысли об обработке ошибок"

Из блога: JS Recipes

> …Существует три основных направления работы с ошибками:
1. Дайте приложению упасть и перезапустите его.
2. Обрабатывайте все возможные ошибоки и никогда не роняйте сервис.
3. Сбалансированный подход между двумя предыдущими.