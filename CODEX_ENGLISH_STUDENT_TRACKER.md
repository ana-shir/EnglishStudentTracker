# Техническое задание для Codex: English Student Tracker

## 1. Задача проекта

Создать статический веб-проект для GitHub Pages, состоящий из двух
страниц:

-   `index.html` --- публичный трекер для просмотра результатов
    учеников.
-   `admin.html` --- закрытая админ-панель для учителя.

Обе страницы должны работать с одним Firebase-проектом
`english-student-tracker`.

Использовать: - Cloud Firestore; - Firebase Authentication; -
Email/Password Authentication; - Firebase JavaScript SDK; - Vanilla
HTML + CSS + JavaScript; - ES Modules; - Firebase SDK через CDN; - без
npm, React, Vue и серверной части; - хостинг --- GitHub Pages.

## 2. Структура проекта

На первом этапе:

``` text
/
├── index.html
└── admin.html
```

Каждый HTML-файл автономный: HTML, CSS внутри `<style>`, JavaScript
внутри `<script type="module">`.

## 3. Firebase-приложения

В Firebase зарегистрированы два Web App:

-   `English Tracker Public` --- использовать Firebase config в
    `index.html`.
-   `English Tracker Admin` --- использовать Firebase config в
    `admin.html`.

В коде оставить заметный комментарий:

``` javascript
// ========================================
// FIREBASE CONFIG
// ВСТАВИТЬ CONFIG ИЗ FIREBASE CONSOLE
// ========================================
```

Не использовать Firebase Admin SDK, service account и серверные
секретные ключи.

## 4. Архитектура Firestore

Корневые коллекции:

``` text
groups
students
```

У каждого ученика вложенная коллекция `lessons`:

``` text
groups
└── {groupId}

students
└── {studentId}
    └── lessons
        ├── lesson_01
        ├── lesson_02
        ├── ...
        └── lesson_16
```

## 5. Документ группы

Коллекция `groups`, документы с Auto ID.

``` javascript
{
    name: "Starter A",
    scheduleDays: [2, 4],
    firstLessonDate: Timestamp,
    lessonDates: [Timestamp, Timestamp, ...],
    createdAt: serverTimestamp()
}
```

`lessonDates` содержит ровно 16 дат.

## 6. Дни недели

Использовать JavaScript-нумерацию:

-   0 --- Sunday / Воскресенье
-   1 --- Monday / Понедельник
-   2 --- Tuesday / Вторник
-   3 --- Wednesday / Среда
-   4 --- Thursday / Четверг
-   5 --- Friday / Пятница
-   6 --- Saturday / Суббота

В интерфейсе показывать русские названия.

## 7. Создание группы

В `admin.html` кнопка `+ Добавить группу`.

Форма: - Название группы. - Расписание: чекбоксы дней недели. - Дата
первого занятия. - Кнопка `Создать группу`.

Валидация: 1. Название обязательно. 2. Выбран минимум один день. 3. Дата
первого занятия обязательна. 4. День недели первой даты должен
присутствовать в `scheduleDays`. 5. При несоответствии показать: «Дата
первого занятия должна соответствовать одному из выбранных дней
расписания».

## 8. Автоматическое создание 16 дат

После создания группы рассчитать даты 16 занятий.

Создать функцию:

``` javascript
generateLessonDates(firstDate, scheduleDays, count = 16)
```

Алгоритм: 1. Первая дата --- урок 1. 2. Добавить её в массив. 3. Идти
дальше по календарю по одному дню. 4. Проверять `date.getDay()`. 5. Если
день входит в `scheduleDays`, добавить дату. 6. Остановиться после 16
дат.

## 9. Карточка группы

Показывать: - название; - расписание; - дату первого занятия; - «16
занятий»; - кнопку `Добавить ученика`.

## 10. Создание ученика

В `admin.html` кнопка `+ Добавить ученика`.

Форма: - Имя ученика. - Группа (select). - Кнопка `Добавить ученика`.

Нельзя создавать ученика без группы.

## 11. Документ ученика

`students/{studentId}` с Auto ID:

``` javascript
{
    name: "Анна",
    groupId: "firebase-group-id",
    totalCoins: 0,
    winnerStickers: 0,
    active: true,
    createdAt: serverTimestamp()
}
```

Ученикам не создавать Firebase Authentication accounts.

## 12. Автоматическое создание уроков

После создания ученика создать подколлекцию `lessons` и документы
`lesson_01` ... `lesson_16`.

Использовать даты из `group.lessonDates`.

Использовать Firestore `writeBatch`.

## 13. Документ урока

``` javascript
{
    number: 1,
    date: Timestamp,
    homework: false,
    behavior: false,
    activity: false,
    games: false,
    coinsEarned: 0
}
```

Аналогично для уроков 1--16.

## 14. Система монет

За урок максимум 4 монеты: - Домашнее задание → `homework` - Поведение →
`behavior` - Активность → `activity` - Победа в играх → `games`

Каждое поле boolean. `true` = одна монета.

## 15. Расчёт монет за урок

Создать:

``` javascript
calculateLessonCoins(lesson)
```

Подсчитать `true` среди четырёх категорий и записать результат 0--4 в
`coinsEarned`.

## 16. Общее количество монет

После изменения монеты пересчитать сумму `coinsEarned` всех 16 уроков и
записать в `students/{studentId}.totalCoins`.

Максимум: 64 монеты.

## 17. Winner Stickers

Один winner-стикер за каждые 30 монет:

``` javascript
winnerStickers = Math.floor(totalCoins / 30);
```

-   0--29 → 0
-   30--59 → 1
-   60--64 → 2

Записать в `winnerStickers`.

## 18. Прогресс к следующему Winner

Не хранить отдельным полем. Вычислять:

``` javascript
const progressCoins = totalCoins % 30;
```

Показывать, например: `🪙 8 / 30 до следующего Winner`.

## 19. ADMIN.HTML --- авторизация

Использовать Firebase Authentication Email/Password:

``` javascript
getAuth
signInWithEmailAndPassword
signOut
onAuthStateChanged
```

До входа показывать только форму: - Email - Пароль - Войти

Регистрацию администратора через сайт не делать.

## 20. Поведение авторизации

До авторизации административный интерфейс скрыт.

После входа: - скрыть login; - показать admin panel.

Использовать `onAuthStateChanged()` для сохранения состояния после
перезагрузки.

## 21. Ошибка входа

Не показывать Firebase error codes. Показывать:
`Неверный логин или пароль.`

## 22. Выход

Кнопка `Выйти`, использующая `signOut(auth)`.

## 23. Главный экран админки

Показывать: - ENGLISH TRACKER; - Панель учителя; - Выйти; - ГРУППЫ +
Добавить группу; - УЧЕНИКИ + Добавить ученика.

## 24. Список учеников

Карточка ученика: - имя; - группа; - количество монет; - Winner × N; -
прогресс N / 30; - кнопка `Открыть`.

Сортировать по имени.

## 25. Подробная карточка ученика

Показывать: - имя; - группу; - totalCoins; - winnerStickers; -
прогресс; - 16 уроков.

## 26. Карточка урока в админке

Показывать номер, дату и 4 интерактивные монеты.

При клике: 1. переключить boolean; 2. пересчитать `coinsEarned`; 3.
сохранить урок; 4. пересчитать `totalCoins`; 5. пересчитать
`winnerStickers`; 6. обновить UI.

## 27. Защита от двойного клика

На время Firestore-запроса блокировать нажатую кнопку.

## 28. Индикатор сохранения

Показывать состояния: - `Сохраняю...` - `Сохранено ✓` -
`Ошибка сохранения`

## 29. Редактирование даты урока

Добавить кнопку редактирования даты конкретного урока. Изменение одной
даты не меняет остальные уроки.

## 30. INDEX.HTML --- публичный трекер

Не использовать Authentication. Только чтение Firestore.

Не должно быть функций редактирования, удаления и административных форм.

## 31. Главный экран публичного трекера

Показывать: - ENGLISH TRACKER; - Наши достижения; - фильтр `Все группы`.

## 32. Публичная карточка ученика

Показывать: - имя; - группу; - монеты; - Winner × N; - прогресс; -
кнопку `Посмотреть уроки`.

## 33. Публичные уроки

Показывать 16 уроков, дату и четыре категории монет. Ничего нельзя
изменять.

## 34. Будущие уроки

Если дата ещё не наступила и монет нет, визуально сделать карточку менее
яркой, сохранив дату.

## 35. Сортировка

Уроки всегда сортировать по `number` ascending.

## 36. Состояния загрузки

Показывать `Загрузка...`.

При пустой базе: - `Пока нет учеников.` - `Пока нет групп.`

## 37. Firebase SDK

Использовать Modular Firebase SDK, а не старый `firebase.firestore()` /
`firebase.auth()`.

Допустимы: `initializeApp`, `getFirestore`, `collection`, `doc`,
`getDocs`, `onSnapshot`, `addDoc`, `setDoc`, `updateDoc`, `writeBatch`,
`serverTimestamp`, `Timestamp`.

## 38. Реальное время

Предпочтительно использовать `onSnapshot()`, чтобы публичный сайт
автоматически отражал изменения из админки.

## 39. Security Rules

Приложение рассчитывает на правила: - `groups`: читать могут все, писать
только ADMIN UID; - `students`: читать могут все, писать только ADMIN
UID; - `students/{student}/lessons`: читать могут все, писать только
ADMIN UID.

Ожидаемая логика:

``` javascript
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {

    function isAdmin() {
      return request.auth != null
             && request.auth.uid == "ADMIN_UID";
    }

    match /groups/{groupId} {
      allow read: if true;
      allow create, update, delete: if isAdmin();
    }

    match /students/{studentId} {
      allow read: if true;
      allow create, update, delete: if isAdmin();

      match /lessons/{lessonId} {
        allow read: if true;
        allow create, update, delete: if isAdmin();
      }
    }
  }
}
```

## 40. Безопасность

`admin.html` не является секретной страницей. Право записи обеспечивает
Firebase Authentication + Firestore Security Rules.

Нельзя хранить пароль в HTML, JS, Firestore, GitHub или localStorage.

Не использовать самодельную проверку пароля.

## 41. Администраторы

Не создавать администраторов из сайта. Учётные записи создаются вручную
через Firebase Console → Authentication → Users.

## 42. Дизайн

Современный дружелюбный интерфейс детского образовательного проекта: -
светлый фон; - крупные карточки; - мягкие скругления; - понятные
кнопки; - крупные цифры; - крупные монеты; - заметный Winner Sticker; -
CSS Grid/Flexbox; - responsive для компьютера, планшета и телефона.

## 43. Иконки

Для MVP: - монета `🪙`; - Winner `🏆`.

CSS сделать так, чтобы позже заменить emoji на PNG/SVG.

## 44. Модальные окна

Для добавления группы и ученика использовать модальные окна.

Закрытие: - ×; - Отмена; - клик по фону; - Escape.

## 45. Удаление

Если реализовано удаление --- обязательно подтверждение. Удаление группы
на первом этапе можно не делать.

## 46. Ошибки

Firestore-операции оборачивать в `try/catch`. Пользователю показывать
понятные сообщения, технические ошибки --- через `console.error()`.

## 47. Основные функции admin.html

Минимально:

``` javascript
loginAdmin()
logoutAdmin()
loadGroups()
createGroup()
generateLessonDates()
loadStudents()
createStudent()
createStudentLessons()
openStudent()
loadStudentLessons()
toggleCoin()
calculateLessonCoins()
recalculateStudentTotals()
formatDate()
showMessage()
showModal()
closeModal()
```

## 48. Основные функции index.html

Минимально:

``` javascript
loadGroups()
loadStudents()
renderStudents()
openStudent()
loadStudentLessons()
renderLessons()
formatDate()
calculateProgress()
```

## 49. Даты и часовые пояса

Не допускать сдвига даты из-за UTC.

Для `<input type="date">`:

``` javascript
const [year, month, day] = value.split("-").map(Number);
const date = new Date(year, month - 1, day, 12, 0, 0);
```

В интерфейсе формат `01.09.2026` или `1 сентября 2026`.

## 50. Пустая база

Firestore может быть изначально пустым.

`admin.html` должен корректно создать первую группу, затем первого
ученика и подколлекцию уроков. Ручное создание коллекций через Firebase
Console не требуется.

## 51. Первый сценарий проверки

1.  Открыть `admin.html`.
2.  Войти по Firebase email/password.
3.  Создать группу `Starter A`.
4.  Выбрать Вторник и Четверг.
5.  Указать `01.09.2026`.
6.  Проверить автоматическое создание 16 дат.
7.  Добавить ученика `Анна` в `Starter A`.
8.  Проверить создание student + 16 lessons.
9.  Открыть ученика.
10. Выдать несколько монет.
11. Открыть `index.html`.
12. Убедиться, что ученик и результаты отображаются.

## 52. GitHub Pages

Оба файла держать в одном репозитории.

Пример:

``` text
https://USERNAME.github.io/english-student-tracker/
https://USERNAME.github.io/english-student-tracker/admin.html
```

Не использовать пути, которые ломаются в repository Pages.

## 53. Пока не реализовывать

Не делать: - регистрацию учеников; - личные кабинеты учеников; - пароли
учеников; - Firebase Storage; - фото; - push notifications; - Cloud
Functions; - Node backend; - рейтинг; - оплату; - рассылки; - Excel; -
PDF; - роли нескольких администраторов; - историю изменений; - игры.

## 54. Конфиденциальность

На публичном сайте не показывать фамилии, телефоны, email, адреса, даты
рождения и другие персональные данные.

Использовать только отображаемое имя, например `Анна`, `Максим`,
`Маша К.`.

## 55. Критерии готовности

Проверить: 1. `index.html` работает. 2. `admin.html` работает. 3.
Firebase инициализируется. 4. Email/password login работает. 5. Неверный
пароль обрабатывается. 6. Logout работает. 7. Без входа административные
функции недоступны. 8. Группа создаётся. 9. 16 дат рассчитываются. 10.
Ученик добавляется. 11. 16 уроков создаются. 12. Все четыре монеты
переключаются. 13. `coinsEarned` пересчитывается. 14. `totalCoins`
пересчитывается. 15. `winnerStickers` пересчитывается. 16. Публичный
tracker показывает результаты. 17. Публичный tracker не редактирует
данные. 18. Данные сохраняются после перезагрузки. 19. Публичный сайт
получает актуальные изменения. 20. Всё работает на GitHub Pages.

## 56. Проверка безопасности

Без входа попытка записи должна получить `permission-denied`.

После входа правильным администратором запись должна работать.

`index.html` реализует только чтение.

## 57. Качество кода

Перед завершением проверить: - HTML; - Firebase imports; - undefined
variables; - обработчики; - console errors; - responsive; - пустую
Firestore; - создание группы; - создание ученика; - четыре монеты; - 30
монет → 1 Winner; - 60 монет → 2 Winner.

Не оставлять TODO, ломающие основную функциональность.

## 58. Результат

Создать: - `index.html` - `admin.html`

После выполнения кратко описать: - где вставляется Firebase config; -
какие функции реализованы; - как запустить локально; - как опубликовать
через GitHub Pages; - как выполнить первую проверку.

Главная цель --- два полностью рабочих HTML-файла в одном
GitHub-репозитории, подключённых к одному Firebase-проекту.
