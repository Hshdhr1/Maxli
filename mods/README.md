# 📚 Документация по созданию модулей для Maxli UserBot

> **Maxli** использует библиотеку [PyMax (maxapi-python)](https://fresh-milkshake.github.io/pymax/) — Python-обёртку для Max Messenger API.
> 
> 📖 Полная документация PyMax: https://fresh-milkshake.github.io/pymax/

## 📋 Содержание

1. [Быстрый старт](#быстрый-старт)
2. [Структура модуля](#структура-модуля)
3. [Maxli API](#maxli-api)
4. [Прямой доступ к PyMax](#прямой-доступ-к-pymax)
5. [Форматирование текста](#форматирование-текста)
6. [Работа с файлами и медиа](#работа-с-файлами-и-медиа)
7. [Конфигурация модулей](#конфигурация-модулей)
8. [Продвинутые возможности](#продвинутые-возможности)
9. [Примеры модулей](#примеры-модулей)

---

## Быстрый старт

### Минимальный модуль

```python
# name: Мой модуль
# version: 1.0.0
# developer: Ваше имя
# id: my_module
# min-maxli: 35

async def hello_command(api, message, args):
    """Простое приветствие."""
    await api.edit(message, "👋 **Привет!**", markdown=True)

async def register(api):
    api.register_command("hello", hello_command)
```

### Загрузка модуля

1. Создайте файл `.py` с кодом модуля
2. Отправьте файл боту с командой `.load`
3. Или положите файл в папку `modules/` и перезапустите бота
4. Используйте команду с префиксом (например `.hello`)

---

## Структура модуля

### Обязательные метаданные

```python
# name: Название модуля
# version: 1.0.0
# developer: Имя разработчика
# id: unique_module_id
# min-maxli: 35
```

| Поле | Описание |
|------|----------|
| `name` | Отображаемое название модуля |
| `version` | Версия модуля (semver) |
| `developer` | Имя разработчика |
| `id` | Уникальный ID (2-32 символа, латиница, цифры, `-`, `_`) |
| `min-maxli` | Минимальная версия Maxli для работы модуля |

### Основные функции

```python
async def register(api):
    """Вызывается при загрузке модуля."""
    api.register_command("cmd", command_handler)
    api.register_watcher(watcher_handler)

async def command_handler(api, message, args):
    """Обработчик команды.
    
    Args:
        api: Объект Maxli API
        message: Объект сообщения (pymax.Message)
        args: Список аргументов команды
    """
    pass

async def watcher_handler(api, message):
    """Вотчер — обрабатывает все входящие сообщения.
    
    Args:
        api: Объект Maxli API
        message: Объект сообщения (pymax.Message)
    """
    pass
```

---

## Maxli API

Maxli предоставляет удобную обёртку над PyMax с дополнительными функциями.

### Работа с сообщениями

```python
# Редактирование сообщения (с поддержкой markdown)
await api.edit(message, "**Жирный** и *курсив*", markdown=True)

# Отправка нового сообщения
chat_id = await api.get_chat_id_for_message(message)
await api.send(chat_id, "Привет!", markdown=True)

# Ответ на сообщение (в последний известный чат)
await api.reply(message, "Ответ", markdown=True)

# Удаление сообщения
await api.delete(message)
await api.delete(message, for_me=True)  # Только у себя
```

### Получение chat_id

```python
# Способ 1: Из атрибута сообщения (если доступен)
chat_id = getattr(message, 'chat_id', None)

# Способ 2: Через API (с fallback)
chat_id = await api.get_chat_id_for_message(message)

# Способ 3: Полный поиск (медленнее)
chat_id = await api.await_chat_id(message)
```

### Реакции

```python
# Установка реакции на сообщение
await api.set_reaction(message, "❤️")
await api.set_reaction(message, "👍", reaction_type="EMOJI")
```

### Информация о пользователях

```python
# Получение имени пользователя
name = await api.get_user_name(user_id)

# Получение полной информации о пользователе
user = await api.get_user_info(user_id)

# Синхронное получение имени отправителя (из кэша)
sender_name = api.get_sender_name(message)
```

### Свойства API

```python
api.BOT_NAME       # Название бота (Maxli)
api.BOT_VERSION    # Версия бота
api.BOT_VERSION_CODE  # Числовой код версии
api.me             # Информация о текущем пользователе
api.client         # Прямой доступ к PyMax клиенту
api.config         # Конфигурация бота
api.LOG_BUFFER     # Буфер логов
```

---

## Прямой доступ к PyMax

Вы можете использовать все возможности библиотеки PyMax напрямую через `api.client`.

> 📖 Документация PyMax: https://fresh-milkshake.github.io/pymax/

### Основные методы PyMax клиента

```python
client = api.client  # Получаем PyMax клиент

# Отправка сообщений
await client.send_message(chat_id=123, text="Привет!")

# Редактирование сообщений
await client.edit_message(chat_id=123, message_id=456, text="Новый текст")

# Удаление сообщений
await client.delete_message(chat_id=123, message_ids=[456, 789])

# Получение истории чата
messages = await client.fetch_history(chat_id=123, limit=50)

# Работа с пользователями
user = await client.get_user(user_id)
users = await client.fetch_users([id1, id2, id3])

# Работа с чатами
chat = await client.get_chat(chat_id)
chats = await client.get_chats()

# Работа с группами
await client.create_group(title="Группа", user_ids=[123, 456])
await client.invite_users_to_group(group_id=123, user_ids=[456])
await client.remove_users_from_group(group_id=123, user_ids=[456])

# Изменение профиля
await client.change_profile(first_name="Имя", last_name="Фамилия")

# Работа с реакциями
await client.add_reaction(chat_id=123, message_id=456, reaction="❤️")
await client.remove_reaction(chat_id=123, message_id=456)
reactions = await client.get_reactions(chat_id=123, message_id=456)

# Работа с каналами
channel = await client.resolve_channel_by_name("channel_name")
await client.join_channel(channel_id=123)

# Закрепление сообщений
await client.pin_message(chat_id=123, message_id=456)
```

### Доступные модели PyMax

```python
from pymax import (
    Message,        # Сообщение
    User,           # Пользователь
    Chat,           # Чат
    Dialog,         # Диалог
    Channel,        # Канал
    Member,         # Участник группы
    Photo,          # Фото
    File,           # Файл
    Me,             # Информация о себе
)
```

### Типы и перечисления

```python
from pymax import (
    ChatType,       # Тип чата (DIALOG, GROUP, CHANNEL)
    MessageType,    # Тип сообщения
    MessageStatus,  # Статус сообщения
    AttachType,     # Тип вложения
    FormattingType, # Тип форматирования (STRONG, EMPHASIZED, UNDERLINE, STRIKETHROUGH)
)
```

### Декораторы для обработчиков

PyMax поддерживает декораторы для регистрации обработчиков событий:

```python
client = api.client

# Обработчик всех сообщений
@client.on_message()
async def handle_all_messages(message):
    print(f"Новое сообщение: {message.text}")

# Обработчик с фильтром
from pymax.filters import filters

@client.on_message(filters.text)
async def handle_text_messages(message):
    print(f"Текстовое сообщение: {message.text}")

# Обработчик редактирования
@client.on_message_edit()
async def handle_edit(message):
    print(f"Сообщение отредактировано: {message.text}")

# Обработчик удаления
@client.on_message_delete()
async def handle_delete(message):
    print(f"Сообщение удалено: {message.id}")

# Обработчик реакций
@client.on_reaction_change()
async def handle_reaction(reaction_info):
    print(f"Реакция изменена: {reaction_info}")
```

---

## Форматирование текста

### Markdown синтаксис

При использовании `markdown=True` поддерживается следующий синтаксис:

| Синтаксис | Результат | Тип |
|-----------|-----------|-----|
| `**текст**` | **жирный** | STRONG |
| `*текст*` | *курсив* | EMPHASIZED |
| `__текст__` | <u>подчёркнутый</u> | UNDERLINE |
| `~~текст~~` | ~~зачёркнутый~~ | STRIKETHROUGH |

### Пример

```python
await api.edit(message, """
**Жирный текст**
*Курсивный текст*
__Подчёркнутый текст__
~~Зачёркнутый текст~~
**Комбинация *разных* стилей**
""", markdown=True)
```

### Программное создание форматирования

Вы можете создавать элементы форматирования программно:

```python
from pymax.formatting import Formatting
from pymax import FormattingType, Element

# Парсинг markdown в элементы
formatter = Formatting()
elements, clean_text = formatter.get_elements_from_markdown("**bold** *italic*")
# elements: [Element(type=STRONG, from_=0, length=4), Element(type=EMPHASIZED, from_=5, length=6)]
# clean_text: "bold italic"

# Создание элемента вручную
element = Element(
    type=FormattingType.STRONG,
    from_=0,
    length=5
)
```

---

## Работа с файлами и медиа

### Отправка файлов

```python
chat_id = await api.get_chat_id_for_message(message)

# Отправка файла
await api.send_file(
    chat_id=chat_id,
    file_path="path/to/file.txt",
    text="Описание файла",
    markdown=True
)
```

### Отправка фото

```python
# Из локального файла
await api.send_photo(
    chat_id=chat_id,
    file_path="path/to/image.jpg",
    text="**Красивое фото!**",
    markdown=True
)

# Из URL
await api.send_photo(
    chat_id=chat_id,
    file_path="https://example.com/image.jpg",
    text="Фото из интернета"
)
```

### Получение файлов из сообщений

```python
if message.attaches:
    for attach in message.attaches:
        file_name = getattr(attach, 'name', 'unknown')
        file_url = getattr(attach, 'url', None)
        file_id = getattr(attach, 'file_id', None)
        
        print(f"Файл: {file_name}")
        print(f"URL: {file_url}")
```

### Получение URL файла по ID

```python
file_url = await api.get_file_url(
    file_id=attach.file_id,
    token=attach.token,
    message_id=message.id,
    chat_id=chat_id
)
```

### Использование PyMax для работы с файлами

```python
from pymax.files import Photo

# Создание объекта Photo
photo = Photo(path="image.jpg")

# Валидация фото
photo_data = photo.validate_photo()  # Returns (format, mime_type)

# Отправка через клиент
await api.client.send_message(
    chat_id=chat_id,
    text="Фото",
    photo=photo
)
```

---

## Конфигурация модулей

### Регистрация настроек

```python
from core.config import register_module_settings, get_module_setting, set_module_setting

async def register(api):
    # Регистрация настроек модуля
    register_module_settings("my_module", {
        "enabled": {
            "default": True,
            "description": "Включить модуль"
        },
        "prefix": {
            "default": "!",
            "description": "Префикс команд"
        },
        "max_length": {
            "default": 100,
            "description": "Максимальная длина сообщения"
        }
    })
```

### Использование настроек

```python
# Получение значения (с fallback)
enabled = get_module_setting("my_module", "enabled", True)
prefix = get_module_setting("my_module", "prefix", "!")

# Установка значения
set_module_setting("my_module", "enabled", False)
```

---

## Продвинутые возможности

### Вотчеры (Watchers)

Вотчеры обрабатывают все входящие сообщения:

```python
async def auto_reply_watcher(api, message):
    """Автоматический ответ на приветствие."""
    text = getattr(message, 'text', '').lower()
    
    # Пропускаем свои сообщения
    if message.sender == api.me.id:
        return
    
    if text in ["привет", "hi", "hello"]:
        await api.reply(message, "👋 Привет!", markdown=True)

async def register(api):
    api.register_watcher(auto_reply_watcher)
```

### Обработка ошибок

```python
async def safe_command(api, message, args):
    """Команда с безопасной обработкой ошибок."""
    try:
        # Ваш код
        result = await some_operation()
        await api.edit(message, f"✅ **Готово:** {result}", markdown=True)
        
    except ValueError as e:
        await api.edit(message, f"⚠️ **Ошибка валидации:** {e}", markdown=True)
        
    except Exception as e:
        await api.edit(message, f"❌ **Ошибка:** {e}", markdown=True)
        # Логирование
        api.LOG_BUFFER.append(f"[{__name__}] Error: {e}")
```

### Асинхронные операции

```python
import asyncio

async def parallel_command(api, message, args):
    """Параллельное выполнение задач."""
    await api.edit(message, "⏳ Выполняю...", markdown=True)
    
    # Параллельное выполнение
    results = await asyncio.gather(
        task1(),
        task2(),
        task3(),
        return_exceptions=True
    )
    
    await api.edit(message, f"✅ Готово: {results}", markdown=True)
```

### Работа с низкоуровневым WebSocket API

```python
from pymax.static.enum import Opcode
from pymax.payloads import SendMessagePayload, SendMessagePayloadMessage

async def low_level_send(api, chat_id, text):
    """Отправка через низкоуровневый API."""
    import time
    
    message_payload = SendMessagePayloadMessage(
        text=text,
        cid=int(time.time() * 1000),
        elements=[],
        attaches=[],
        link=None
    )
    
    payload = SendMessagePayload(
        chat_id=chat_id,
        message=message_payload,
        notify=True
    ).model_dump(by_alias=True)
    
    data = await api.client._send_and_wait(
        opcode=Opcode.MSG_SEND,
        payload=payload
    )
    
    return data
```

---

## Примеры модулей

### Модуль с настройками

```python
# name: Приветствие
# version: 1.0.0
# developer: Example
# id: greeter
# min-maxli: 35

from core.config import register_module_settings, get_module_setting

async def greet_command(api, message, args):
    """Приветствует с настраиваемым текстом."""
    text = get_module_setting("greeter", "message", "👋 Привет!")
    await api.edit(message, text, markdown=True)

async def set_greeting_command(api, message, args):
    """Устанавливает текст приветствия."""
    if not args:
        await api.edit(message, "⚠️ Укажите текст приветствия", markdown=True)
        return
    
    from core.config import set_module_setting
    new_text = " ".join(args)
    set_module_setting("greeter", "message", new_text)
    await api.edit(message, f"✅ Установлено: **{new_text}**", markdown=True)

async def register(api):
    register_module_settings("greeter", {
        "message": {
            "default": "👋 Привет!",
            "description": "Текст приветствия"
        }
    })
    api.register_command("greet", greet_command)
    api.register_command("setgreet", set_greeting_command)
```

### Модуль загрузки файлов

```python
# name: Файловый менеджер
# version: 1.0.0
# developer: Example
# id: file_manager
# min-maxli: 35

import os

async def upload_command(api, message, args):
    """Загружает файл в чат."""
    if not args:
        await api.edit(message, "⚠️ Укажите путь к файлу", markdown=True)
        return
    
    file_path = " ".join(args)
    
    if not os.path.exists(file_path):
        await api.edit(message, f"❌ Файл не найден: **{file_path}**", markdown=True)
        return
    
    chat_id = await api.get_chat_id_for_message(message)
    await api.edit(message, "⏳ Загружаю файл...", markdown=True)
    
    await api.send_file(
        chat_id=chat_id,
        file_path=file_path,
        text=f"📁 **{os.path.basename(file_path)}**",
        markdown=True
    )
    
    await api.delete(message)

async def register(api):
    api.register_command("upload", upload_command)
```

### Модуль с использованием PyMax напрямую

```python
# name: История чата
# version: 1.0.0
# developer: Example
# id: chat_history
# min-maxli: 35

async def history_command(api, message, args):
    """Показывает последние сообщения чата."""
    limit = int(args[0]) if args else 5
    limit = min(limit, 20)  # Максимум 20
    
    chat_id = await api.get_chat_id_for_message(message)
    await api.edit(message, "⏳ Загружаю историю...", markdown=True)
    
    # Используем PyMax напрямую
    messages = await api.client.fetch_history(chat_id=chat_id, limit=limit)
    
    result = f"📜 **Последние {len(messages)} сообщений:**\n\n"
    for i, msg in enumerate(messages, 1):
        text = getattr(msg, 'text', '')[:50]
        sender = msg.sender
        result += f"{i}. [{sender}] {text}...\n"
    
    await api.edit(message, result, markdown=True)

async def register(api):
    api.register_command("history", history_command)
```

### Модуль-вотчер

```python
# name: Анти-спам
# version: 1.0.0
# developer: Example
# id: antispam
# min-maxli: 35

from core.config import register_module_settings, get_module_setting

# Кэш сообщений для отслеживания спама
message_cache = {}

async def antispam_watcher(api, message):
    """Обнаруживает спам."""
    # Пропускаем свои сообщения
    if message.sender == api.me.id:
        return
    
    if not get_module_setting("antispam", "enabled", True):
        return
    
    sender = message.sender
    text = getattr(message, 'text', '')
    
    # Простая проверка на повторяющиеся сообщения
    if sender in message_cache:
        if message_cache[sender] == text:
            print(f"[antispam] Обнаружен спам от {sender}")
    
    message_cache[sender] = text

async def toggle_command(api, message, args):
    """Включает/выключает антиспам."""
    from core.config import set_module_setting
    
    current = get_module_setting("antispam", "enabled", True)
    set_module_setting("antispam", "enabled", not current)
    
    status = "включён" if not current else "выключен"
    await api.edit(message, f"🛡️ Антиспам **{status}**", markdown=True)

async def register(api):
    register_module_settings("antispam", {
        "enabled": {
            "default": True,
            "description": "Включить антиспам"
        }
    })
    api.register_command("antispam", toggle_command)
    api.register_watcher(antispam_watcher)
```

---

## Полезные ссылки

- 📖 **PyMax документация:** https://fresh-milkshake.github.io/pymax/
- 📦 **PyMax на PyPI:** `pip install maxapi-python`
- 🐙 **Исходный код Maxli:** GitHub репозиторий

## Полезные советы

1. ✅ Всегда используйте `markdown=True` для форматированного текста
2. ✅ Оборачивайте код в `try/except` для обработки ошибок
3. ✅ Используйте docstring для документирования команд
4. ✅ Проверяйте `min-maxli` для совместимости
5. ✅ Используйте `api.client` для доступа ко всем возможностям PyMax
6. ✅ Логируйте ошибки в `api.LOG_BUFFER`
7. ✅ Очищайте временные файлы после использования