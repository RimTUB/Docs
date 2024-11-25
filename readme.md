# Документация для разработчиков

Разработчики могут писать свои модули для RimTUB. Тут будет вся необходимая информация.

> Модуль — это отдельный компонент или блок кода, который расширяет функционал юзербота, добавляя ему новые команды и возможности. Каждый модуль обычно отвечает за выполнение конкретной задачи, будь то автоматизация действий, интеграция с внешними сервисами, управление сообщениями или предоставление информации. Модули легко добавляются и удаляются, что позволяет гибко настраивать работу юзербота в зависимости от потребностей пользователя.

## Содержание
- [Техническая информация](#tech_info)
- [Структура модуля](#structure)
- [Упаковка и отправка модулей](#module_packing)
- [Modify Pyrogram Client](#ModifyPyrogramClient)
- [DataBase](#DataBase)
    + [Структура](#DataBase.structure)
    + [Вставка](#DataBase.set)
    + [Получение](#DataBase.get)
    + [Удаление](#DataBase.delete)
- [LocalStorage](#LocalStorage)
    + [Вставка](#LocalStorage.set)
    + [Получение](#LocalStorage.get)
    + [Удаление](#LocalStorage.delete)
- [HelpList](#HelpList)
- [обработчики, фильтры и команды | cmd](#handlers-filters-cmd)
    + [account_filter](#account_filter)
    + [text_filter](#text_filter)
    + [thread_filter](#thread_filter)

<a id='tech_info'></a>

## Техническая информация
RimTUB использует Pyrogram fork by @KurimuzonAkuma как основной способ взаимодействия с Telegram MTProto API. Вместе с Pyrogram используется pyromod для расширения функционала. Для работы с BotAPI используется pyTelegramBotAPI ❤️

<a id='structure'></a>

## Структура модуля
Модуль в проекте - это папка с python файлами в директории `plugins`.
Папку следует называть в CamelCase. Например, `ExampleModule`.
В папке модуля должен быть файл `__init__.py`, чтобы модуль был импортируемым.
Ты можешь создавать свои файлы для удобства.
```py
.
└───plugins
    ├───Calculator
    │       __init__.py
    │
    ├───Fishh
    │       __init__.py
    │       worker.py
    │
    └───ExampleModule
        │   __init__.py
        │   utils.py
        │   database.db
        │   file.txt
        │   .rimtubignore # см. Упаковка и отправка модулей
        │
        └───dir
            | ...
        

```
Модуль должен содержать функцию `async def main(app)` в которой загружаются хендлеры. 
Если ты используешь дополнительные библиотеки то их следует перечислить в переменной `__libs__` для их автоматической установке.\
Пример:
```python
__libs__ = ['pytimeparse2']
```
Если библиотека устанавливается одним названием, а импортируется другой, то следует сделать так:
```python
# импортируется так ⬎      |        ⬐ устанавливается так 
__libs__   =   [['telebot', 'pyTelegramBotAPI']]
```

<a id='module_packing'></a>

## Упаковка и отправка модулей
Модули, для удобной передачи и хранения можно упаковать в файл `.rimtub-module`. Это сжатый zip архив. Для упаковки и распаковки используй `utils.pack_module` и `utils.unpack_module` или воспользуйся встроенными командами `.sm` и `.dmf` соответственно.
Если ты не хочешь чтобы упаковывались все файлы, то создай файл `.rimtubignore` в папке модуля. Это аналог gitignore.
```gitignore
*.db
*.txt
!file.txt
dir/*
```
упакованный файл можно просто кому-либо передать.

## Modify Pyrogram Client
Модифицированный клиент расширяет функционал, и вмещает в себе удобные инструменты для разработки


## DataBase
RimTUB имеет свою БД.

<a id='DataBase.structure'></a>

### Структура
Структура БД достаточно примитивная, но удобная для использования в ЮзерБотах. Данные хранятся в формате  модуль + ключ = значение.
БД хранит только простые типы и структуры данных: `str`, `int`, `float`, `bool`, `list` и `dict`.

<a id='DataBase.set'></a>

### Вставка
```python
await app.db.set(__package__, 'var', 'val')
# ╭⎯──────────────────╯         │      │
# │  ╭⎯─────────────────────────╯      │
# │  │ Значение: str, int, float, bool, list и dict.
# │ Имя переменной: str 
# Имя модуля: str. Рекомендуется использовать __package__
```

<a id='DataBase.get'></a>

### Получение
Аналогичная картина
```python
# получение одной переменной:
await app.db.get(__package__, 'var', 0)
# ╭⎯──────────────────╯         │    │
# │  ╭⎯─────────────────────────╯    │
# │  │ Значение по умолчанию. Вернется если значение не найдено
# │ Имя переменной: str 
# Имя модуля: str.

# получение всех переменных:
await app.db.getall(__package__, 0) # -> Словарь  переменная: значение
# ╭⎯─────────────────────╯       │
# │ Значение по умолчанию. Вернется если значение не найдено
# Имя модуля: str.

```

<a id='DataBase.delete'></a>

### Удаление
```python
await app.db.remove(__package__, 'var')
#            ╭⎯──────────╯         │
#      Имя модуля: str.    Имя переменной: str 
```


## LocalStorage
Иногда не требуется долгосрочное хранение, а только временное. Для этого есть local storage. В нем данные сохраняется до перезагрузки юб.

LocalStorage - это немножко доработанный словарь.
В хранилище нет разделения на модули. Поэтому лучше переменные назвать как-то так: `f"{__package__}.var"`

<a id='LocalStorage.set'></a>

### Вставка
```py
app.st.set('var', 'val')
#         ╭─⎯╯      ╰─────────╮
# Имя переменной: Any     Значение
```

<a id='LocalStorage.get'></a>

### Получение
```py
app.st.get('var', 0)
#         ╭─⎯╯    ╰──────────╮
# Имя переменной: Any     Значение по умолчанию
```

<a id='LocalStorage.delete'></a>

### Удаления
```py
del app.st['var']
#         ╭─⎯╯
# Имя переменной: Any
```


## HelpList
Хочешь чтобы твой модуль показывался в `.help`? Смотри как это сделать.
```python
from utils import * # импортируем утилсы

helplist.add_module( # Добавляем модуль в хелплист
    Module( 
        __package__, # Название модуля. Должен совпадать с именем папки модуля. Рекомендуется использовать __package__
        description="Это пример модуля", # Описание модуля (опционально)
        author="@RimMirK", # Автор, разработчик модуля (опционально)
        version='1.0' # Версия (опционально)
    ).add_command( # Сразу добавляем команду в модуль
        Command(['cmd', 'command'], [ ], 'Тестовая команда без аргументов')
#                      │             │                 │
#          ╭⎯──────────╯             │         Описание команды
#          │       Аргументы, которые принимает команда
# Список команд, которые вводит пользователь
    ).add_command( # Добавляем следующую команду
        Command(['arg'], [Argument('Арг 1'), Arg('Арг 2', False)], 'Команда с аргументами')
# ╭⎯───────────────╯     │   │  ╭─────┼───────╯     │       │                 │
# │ ╭⎯───────────────────╯   │  │     ├─────────────╯       │                 │
# │ │ ╭─⎯────────────────────╯  │     │                     │                 │
# │ │ │ ╭─⎯─────────────────────╯     │                     │          Описание команды 
# │ │ │ │ ╭─⎯─────────────────────────╯                     │         
# │ │ │ │ │ ╭─⎯─────────────────────────────────────────────╯
# │ │ │ │ │ Указывает на то, что аргумент не обязательный
# │ │ │ │ Название аргумента
# │ │ │ Тоже самое что Argument
# │ │ Аргумент
# │Аргументы, которые принимает команда
# Список команд, которые вводит пользователь
        
    ).add_feature( # Добавляем возможность. Возможность - это функционал, который не имеет команд.
        Feature("Ничегошка", "Ничего не делает за Вас")
#                    │                   │         
#                Название             Описание
    )
)
```

<a id='handlers-filters-cmd'></a>

## Обработчики, фильтры и команды | cmd

Для регистрации обработчика команды следует использовать cmd. Пример ниже
```py
from utils import *


async def main(app: Client):

    cmd = app.cmd(app.get_group(__package__)) # запомни эту строку как правило. 


    @cmd(['mycommand'])
    async def _mycommand(_, msg: M):
        pass
```

При регистрации своих обработчиков нужно указывать полученную группу
```py
from utils import *


async def main(app: Client):

    group = app.get_group(__package__)

    @app.on_message(filters.user(), group=group)
    async def _from_user(_, msg: M):
        pass
```

Также в RimTUB есть несколько дополнительных фильтров которые могут помочь в разработке твоих крутых модулей

- 
    ## account_filter
    Так-как RimTUB поддерживает мультиаккаунт иногда нужно зарегистрировать хендлер только для одного аккаунта. Тут идет в помощь `account_filter`.
    ### Parameters
    `id` : _int_ \
    ID клиента: `ModifyPyrogramClient.me.id`

    ### Returns
    **Filter**\
    Фильтр, который проверяет, совпадает ли идентификатор пользователя с указанным.

    ### Пример
    ```py
    from utils import *

    @app.on_message(account_filter(466040514), group=group)
    ```

-   
    ## text_filter
    Проверяет сообщения на определенный текст

    ### Parameters
    `text` : _str|List[str]_ \
    Текст или список текстов для фильтрации.

    ### Returns
    **Filter**\
    Фильтр, который проверяет, совпадает ли текст сообщения с указанным.
    

    ### Пример
    ```py
    from utils import *

    @app.on_message(text_filter('Именно этот текст'), group=group)
    ```

-   
    ## thread_filter
    Проверяет сообщения на топик

    ### Parameters
    `message_thread_id` : _int_\
    ID топика: `Message.message_thread_id`.

    ### Returns
    **Filter**\
    Фильтр, который проверяет, совпадает ли идентификатор потока с указанным.
    

    ### Пример
    ```py
    from utils import *
    from pyrogram import filters

    @app.on_message(filters.text & message_thread_filter(2), group=group)
    ```


    ***

    ## utils

    ### install_log

    Устанавливает обработчики логирования и настройки для указанного логгера.

    #### Parameters:
    `logger` : _logging.Logger_\
    Логгер, для которого нужно установить обработчики.\
    `bot` : _bool_\
    Флаг, указывающий, использовать ли уровень логирования для бота (BOT_LOGGING_LEVEL).
    
    #### Returns:
    _logging.Logger_ : Настроенный логгер с добавленными обработчиками.

    #### Example:
    ```py
    from utils import *

    my_logger = logging.getLogger('example')
    my_logger = install_log(my_logger)
    my_logger.info("hello?")
    ```
    
    *** 

    ### format_callback

    Форматирует строку обратного вызова, используя ID пользователя или клиент и ID функции или объект функции. Нужно для правильного составления кнопок.\
    Подробнее в разделе про кнопки

    #### Parameters:
    `user`: _int | ModifyPyrogramClient | Client_ \
    ID клиента или объект клиента, который инициировал запрос.\
    `func`: _int | Awaitable_\
    ID функции или вызываемая асинхронная функция.\
    
    #### Returns:
    _str_ : Строка в формате `{user_id}:{func_id}` для подстановки
    
    #### Example:
    ```py
    from utils import *
    from telebot.types import InlineKeyboardButton as IB

    async def click(...): ...

    IB("нажми меня", callback_data=format_callback(app, click))
    ```

    ## HTML
    ### escape
    Экранирует специальные HTML символы в строке.

    #### Parameters:
    `text` : _str_\
    Текст для экранирования.

    #### Returns:
    _str_ : Экранированный текст.

    ***

    ### format_tag
    Генерирует HTML тег.

    #### Parameters
    `tag_name` : _str_\
    Имя тега. Например "a" или "div".

    `content` : _str_\
    Содержимое тега, defaults to "".

    `escape_content` : _bool_\
    Нужно ли экранировать контент, defaults to True.

    `close_tag` : _bool_\
    Нужен ли закрывающий тег, defaults to True.

    _kwargs_ : Атрибуты тега. Названия атрибутов можно указывать с "_".

    #### Returns
    _str_ : Сгенерированный HTML тег.

    *** 

    ### code, b, i, u, s, spoiler
    Создают соответствующие HTML теги\
    - code: `код`
    - b: **жирный текст**
    - i: _курсив_
    - u: <u>подчеркнутый</u>
    - s: <s>зачеркнутый</s>
    - spoiler: спойлер. Нужно кликнуть чтобы прочитать

    #### Parameters
    `text` : _str_\
    Текст для размещения в теге.

    `escape_html` : _bool_\
    Нужно ли экранировать HTML, по умолчанию True.

    #### Returns:
    _str_ : Генерирует HTML тег.

    ##### Example:
    ```py
    from utils import *

    await msg.edit(b('Hello,') +' '+ i("world!"))
    ```

    ***

    ### a
    Создает HTML тег для ссылки.

    #### Parameters
    `text` : _str_\
    Текст ссылки.

    `url` : _str_\
    URL для ссылки.

    `escape_html` : _bool_\
    Нужно ли экранировать HTML, defaults to True.

    #### Returns
    _str_ : Сгенерированный HTML тег для ссылки.

    #### Example:
    

    ***

    ### blockquote, bq



