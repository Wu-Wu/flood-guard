Описание системы
================

## Введение

TODO


## Требования

Для функционирования системы необходимо... TODO

Полная спецификация использованного оборудования приведена в файле [SPECIFICATION.md](SPECIFICATION.md).


## Возможности

### Контроль нескольких групп датчиков протечки

Система позволяет контролировать несколько групп датчиков, разнесённых территориально, например по разным помещениям, но запитываемых от одних и тех же стояков ХВС/ГВС.

Группа датчиков состоит из одного или более датчиков одного типа включенных параллельно.

В данный момент, используется лишь 3-и входа контроллера из 8-ми, что позволяет довольно легко расширить количество групп датчиков. Логически группы поделены на:

1. датчики в туалете;
2. датчики в кухне;
3. датчики в ванной комнате.

Для отображения статуса тревоги используется индикатор **ALM** (![red](i/red.png)) с необходимой программой включения, позволяющий визуально понять состояние.

### Техническое обслуживание приводов кранов

Для приводов кранов проводится периодическое (раз в неделю) обслуживание для предотвращения их "закисания". Константами в коде логики можно указать день недели, час и минуту начала обслуживания.

	const MNT_HOUR          = 4;            # 04:07
	const MNT_MINUTE        = 7;            # 04:07
	const MNT_DOW           = 5;            # пятница

Техническое обслуживание пропускается, если в момент его наступления имется протечка (сработал датчик(и) в одной или нескольких группах.

Техническое обслуживание прекращается, если в момент его работы сработал один или несколько датчиков.

Для отображения статуса обслуживания используется индикатор **MNT** (![red](i/blue.png)) с необходимой программой включения, позволяющий визуально понять состояние.

### Отложенное открытие кранов

В случае устранения протечки, сигнал на открытие кранов подаётся не сразу, а спустя некоторое время, задаваемое константой

	const PERIOD_GRACE      = PERIOD_30;    # отсрочка 30 сек

На индикаторе **ALM** (![red](i/red.png)) отображается соответствующая программа. По завершении периода, подаётся сигнал на открытие кранов.

### Режим тишины

Для продотвращения получения звонков/сообщений от контроллера, предусмотрены пара констант, позволяющих указать интервал (час начала и час окончания), когда звонки/сообщения будут подавляться. При этом все необходимые действия будет выполнены.

	const MUTE_FROM         = 22;           # > 22:00
	const MUTE_TILL         = 9;            # < 09:00

Это касается системных событий (отключение/восстановление внешнего питания, разряд батареи и прочие).

### Ручной режим управления приводами кранов

Кроме автоматического управления открытием/закрытием кранов, имеется возможность управлять ими в ручном режиме. Для этого служит пара переключателей:

* **MODE** - выбор режима ("M" / "A")
* **VALVE** - управление кранами ("O" / "C").

Индикация состояния кранов (открыты или закрыты) отображается в обоих режимах соответствующими индикаторами.

При переводе в ручной режим управления, приводы кранов не будет реагировать на команды контроллера. Он будет работать в штатном режиме (реагировать на изменения состояния датчиков протечки, выполнять техническое обслуживание и пр.), однако шаровые приводы кранов не будет исполнять его команды.


## События

TODO


## Сценарии логики

### Общие приципы

Сценарии, основанные на коде Морзе запускаются и останавливаются по событиям. Приняты следующие длительности интервалов для кодирования знаков.

Интервал                        | Такты | Время, мс | Элемент
------------------------------- | ----- | --------- | -------
Элемент знака "тире"            | 8     | 800       | ![dash](i/dash_red.png)
Элемент знака "точка"           | 4     | 400       | ![dot](i/dot_red.png)
Пауза между элементами знака    | 2     | 200       |
Пауза до начала повтора знака   | 8     | 800       |

Для каждого сценария, если дополнительно не указано иное, необходимо установить параметры (галочки):

* **бесконечное** число повторений;
* **высокий** начальный уровень сигнала;



### Сценарий \#6 (`$SCENARIO6`)

Код Морзе буквы **W** (латиница): **W**orking

![dot](i/dot_blue.png)![dash](i/dash_blue.png)![dash](i/dash_blue.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6
------- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор технического обслуживания приводов кранов **MNT** (![blue](i/blue.png)).

##### Фактической смысл

Техническое обслуживание приводов кранов проходит в штатном режиме.



### Сценарий \#7 (`$SCENARIO7`)

Код Морзе буквы **S** (латиница): **S**kipping

![dot](i/dot_blue.png)![dot](i/dot_blue.png)![dot](i/dot_blue.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6
------- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 4   | 10

##### Использование

Индикатор технического обслуживания приводов кранов **MNT** (![blue](i/blue.png)).

##### Фактической смысл

Техническое обслуживание приводов кранов отменено из-за активности датчиков протечки.



### Сценарий \#8 (`$SCENARIO8`)

Код Морзе буквы **D** (латиница): wrong **D**ay of week

![dash](i/dash_blue.png)![dot](i/dot_blue.png)![dot](i/dot_blue.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6
------- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 4   | 2   | 4   | 10

##### Использование

Индикатор технического обслуживания приводов кранов **MNT** (![blue](i/blue.png)).

##### Фактической смысл

Техническое обслуживание приводов кранов не будет выполняться из-за неверного дня недели.



### Сценарий \#9 (`$SCENARIO9`)

Код Морзе буквы **X** (латиница): e**X**iting

![dash](i/dash_blue.png)![dot](i/dot_blue.png)![dot](i/dot_blue.png)![dash](i/dash_blue.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8
------- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 4   | 2   | 4   | 2   | 8   | 10

##### Использование

Индикатор технического обслуживания приводов кранов **MNT** (![blue](i/blue.png)).

##### Фактической смысл

Техническое обслуживание приводов кранов прекращено из-за срабатывания датчиков протечки.



### Сценарий \#10 (`$SCENARIO10`)

Код Морзе буквы **G** (латиница): **G**race period

![dash](i/dash_red.png)![dash](i/dash_red.png)![dot](i/dot_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6
------- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 8   | 2   | 4   | 10

##### Использование

Индикатор тревоги **ALM** (![blue](i/red.png)).

##### Фактической смысл

Действует режим отсрочки запуска приводов кранов на открытие.



### Сценарий \#11 (`$SCENARIO11`)

Код Морзе цифры **1** (один)

![dot](i/dot_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 8   | 2   | 8   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![blue](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков туалета.



### Сценарий \#12 (`$SCENARIO12`)

Код Морзе цифры **2** (два)

![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 8   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![blue](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков кухни.



### Сценарий \#13 (`$SCENARIO13`)

Код Морзе цифры **3** (три)

![dot](i/dot_red.png)![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 4   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![blue](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков ванной комнаты.


## Лицензия

MIT

[http://opensource.org/licenses/MIT](http://opensource.org/licenses/MIT)
