# WatchTogether — Инструкция по настройке и деплою

## 1. Настройка Firebase

### 1.1. Создание проекта

1. Перейдите на [Firebase Console](https://console.firebase.google.com/).
2. Нажмите **"Создать проект"** (или выберите существующий).
3. Следуйте шагам мастера создания (можно отключить Google Analytics).
4. Дождитесь создания проекта.

### 1.2. Включение Firestore Database

1. В левой панели навигации нажмите **"Firestore Database"**.
2. Нажмите **"Создать базу данных"**.
3. Выберите **"Запустить в тестовом режиме"** (это позволит читать/писать без аутентификации на 30 дней).
   - **Важно:** Для продакшена настройте правила безопасности.
4. Выберите регион (рекомендуется `eur3` — Европа).
5. Нажмите **"Готово"**.

### 1.3. Настройка правил Firestore

1. Перейдите во вкладку **"Правила"** в Firestore.
2. Замените правила на следующие (для тестового режима):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{roomId} {
      allow read, write: if true;
    }
  }
}
```

3. Нажмите **"Опубликовать"**.

### 1.4. Получение конфигурации Firebase

1. В Firebase Console нажмите на **шестерёнку** (настройки проекта) рядом с "Обзор проекта".
2. Перейдите во вкладку **"Общие"** → прокрутите вниз до **"Ваши приложения"**.
3. Нажмите иконку **Web** (`</>`).
4. Зарегистрируйте приложение (введите любое название, например `watchtogether`).
5. Скопируйте объект `firebaseConfig` — он выглядит так:

```javascript
const firebaseConfig = {
    apiKey: "AIza...",
    authDomain: "your-project.firebaseapp.com",
    projectId: "your-project",
    storageBucket: "your-project.appspot.com",
    messagingSenderId: "123456789",
    appId: "1:123456789:web:abcdef"
};
```

### 1.5. Вставка конфигурации в код

Откройте файл `room.html` и найдите строку ~340:

```javascript
const firebaseConfig = {
    apiKey: "ВАШ_API_KEY",
    authDomain: "ВАШ_PROJECT_ID.firebaseapp.com",
    projectId: "ВАШ_PROJECT_ID",
    storageBucket: "ВАШ_PROJECT_ID.appspot.com",
    messagingSenderId: "ВАШ_SENDER_ID",
    appId: "ВАШ_APP_ID"
};
```

Замените значения на свои из Firebase Console.

---

## 2. Деплой на GitHub Pages

### 2.1. Создание репозитория

1. Перейдите на [GitHub](https://github.com/).
2. Нажмите **"New repository"** (Новый репозиторий).
3. Назовите его, например `watchtogether`.
4. Выберите **Public**.
5. Нажмите **"Create repository"**.

### 2.2. Загрузка файлов

**Вариант A: Через веб-интерфейс**

1. В репозитории нажмите **"uploading an existing file"**.
2. Перетащите файлы `index.html` и `room.html`.
3. Нажмите **"Commit changes"**.

**Вариант B: Через Git (командная строка)**

```bash
cd E:\google\watchtogether
git init
git add .
git commit -m "Initial commit: WatchTogether app"
git branch -M main
git remote add origin https://github.com/ВАШ_USERNAME/watchtogether.git
git push -u origin main
```

### 2.3. Включение GitHub Pages

1. В репозитории перейдите во вкладку **"Settings"**.
2. В левой панели найдите раздел **"Pages"**.
3. В разделе **"Source"** выберите:
   - Branch: `main`
   - Folder: `/ (root)`
4. Нажмите **"Save"**.
5. Через 1-2 минуты сайт будет доступен по адресу:
   ```
   https://ВАШ_USERNAME.github.io/watchtogether/
   ```

---

## 3. Использование

### 3.1. Создание комнаты

1. Откройте главную страницу приложения.
2. Введите свой ник (или оставьте "Гость").
3. Нажмите **"Создать комнату"**.
4. Скопируйте ID комнаты и отправьте друзьям.

### 3.2. Вход в комнату

1. Друг откроёт ссылку `https://ВАШ_USERNAME.github.io/watchtogether/room.html?id=ABC123`
   или введёт ID комнаты на главной странице.
2. Все участники увидят одно и то же видео синхронно.

### 3.3. Поддерживаемые видео

| Платформа | Пример ссылки |
|-----------|---------------|
| YouTube | `https://www.youtube.com/watch?v=dQw4w9WgXcQ` |
| YouTube (короткая) | `https://youtu.be/dQw4w9WgXcQ` |
| Vimeo | `https://vimeo.com/148751763` |
| Прямой MP4 | `https://example.com/video.mp4` |

---

## 4. Структура данных в Firestore

Каждая комната хранится в коллекции `rooms` и имеет следующую структуру:

```
rooms/{roomId}
├── videoUrl: string           // Текущая ссылка на видео
├── playerState: object
│   ├── isPlaying: boolean     // Воспроизводится ли видео
│   ├── currentTime: number    // Текущая позиция (секунды)
│   └── timestamp: number      // Время обновления (ms)
├── messages: array            // Сообщения чата
│   └── [{ user, text, timestamp }]
├── participants: object       // Участники комнаты
│   └── { userId: { nickname, lastSeen } }
└── createdAt: timestamp       // Время создания комнаты
```

---

## 5. Устранение проблем

### Видео не загружается

- Убедитесь, что ссылка корректна.
- YouTube и Vimeo должны быть публичными (не приватными).
- Для MP4 убедитесь, что сервер отдаёт правильный `Content-Type: video/mp4`.

### Синхронизация не работает

- Проверьте консоль браузера (F12) на ошибки Firebase.
- Убедитесь, что Firestore включён и правила позволяют запись.
- Проверьте, что `firebaseConfig` в `room.html` правильный.

### Чат не обновляется

- Firestore в тестовом режиме работает без аутентификации.
- Если правила изменены — убедитесь, что `allow read, write: if true;` установлено.

---

## 6. Безопасность (для продакшена)

Текущие правила Firestore открыты для всех. Для продакшена рекомендуется:

1. Включить **Firebase Authentication** (анонимный вход или через Google).
2. Обновить правила:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /rooms/{roomId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
  }
}
```

3. Ограничить размер сообщений и частоту обновлений.

---

## 7. Лицензия

Этот проект создан в образовательных целях. Используйте свободно!
