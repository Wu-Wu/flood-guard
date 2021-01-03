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

При переводе в ручной режим управления, приводы кранов не будет реагировать на команды контроллера. Он будет работать в штатном режиме (реагировать на изменения состояния датчиков протечки, выполнять техническое обслуживание и пр.), однако шаровые приводы кранов не будут исполнять его команды.

## Приципиальные схемы

### Индикатор состояния кранов

Индикатор располагается в месте коммутации датчиков и кранов (скорее всего туалетная комната) и служит для визуализации состояния приводов кранов (предполагаемое состояние). Он исключает необходимость хождения к электрощиту, где располагаются основные индикаторы состояния.

Схема индикатора доступна в формате DipTrace Schematic Capture v3.0+: `schemes/indicator_unit.dch`.

Может быть реализована как поверхностным монтажом (Surface Mount Technology), так и классическим (Through-hole Technology). В таблице компонентов (на схеме) приведены ориентировочные аналоги компонентов для обоих технологий.

Принципиальная схема индикатора
![схема индикатора](schemes/indicator_unit.png)

Индикатор питается напряжением `+5V`, которое необходимо подать на вывод 1 разъёма. Вывод 2 разъёма должен быть подключен к проводу **OUT** привода крана.


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



### Сценарий \#1 (`$SCENARIO1`)

Код Морзе цифры **1** (один)

![dot](i/dot_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 8   | 2   | 8   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 1.



### Сценарий \#2 (`$SCENARIO2`)

Код Морзе цифры **2** (два)

![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 8   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 2.



### Сценарий \#3 (`$SCENARIO3`)

Код Морзе цифры **3** (три)

![dot](i/dot_red.png)![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dash_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 4   | 2   | 8   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 3.



### Сценарий \#4 (`$SCENARIO4`)

Код Морзе цифры **4** (четыре)

![dot](i/dot_red.png)![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dot_red.png)![dash](i/dash_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 4   | 2   | 4   | 2   | 8   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 4.



### Сценарий \#5 (`$SCENARIO5`)

Код Морзе цифры **5** (пять)

![dot](i/dot_red.png)![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dot_red.png)![dash](i/dot_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 4   | 2   | 4   | 2   | 4   | 2   | 4   | 2   | 4   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 5.



### Сценарий \#6 (`$SCENARIO6`)

Код Морзе цифры **6** (шесть)

![dot](i/dash_red.png)![dot](i/dot_red.png)![dot](i/dot_red.png)![dash](i/dot_red.png)![dash](i/dot_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 4   | 2   | 4   | 2   | 4   | 2   | 4   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 6.



### Сценарий \#7 (`$SCENARIO7`)

Код Морзе цифры **7** (семь)

![dot](i/dash_red.png)![dot](i/dash_red.png)![dot](i/dot_red.png)![dash](i/dot_red.png)![dash](i/dot_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 8   | 2   | 4   | 2   | 4   | 2   | 4   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 7.



### Сценарий \#8 (`$SCENARIO8`)

Код Морзе цифры **8** (восемь)

![dot](i/dash_red.png)![dot](i/dash_red.png)![dot](i/dash_red.png)![dash](i/dot_red.png)![dash](i/dot_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10
------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 8   | 2   | 8   | 2   | 4   | 2   | 4   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Сработал датчик протечки из группы датчиков 8.



### Сценарий \#10 (`$SCENARIO10`)

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



### Сценарий \#11 (`$SCENARIO11`)

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



### Сценарий \#12 (`$SCENARIO12`)

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



### Сценарий \#13 (`$SCENARIO13`)

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



### Сценарий \#14 (`$SCENARIO14`)

Код Морзе буквы **G** (латиница): **G**race period

![dash](i/dash_red.png)![dash](i/dash_red.png)![dot](i/dot_red.png)

##### Столбцы тактов

столбец | 1   | 2   | 3   | 4   | 5   | 6
------- | --- | --- | --- | --- | --- | ---
такты   | 8   | 2   | 8   | 2   | 4   | 10

##### Использование

Индикатор тревоги **ALM** (![red](i/red.png)).

##### Фактической смысл

Действует режим отсрочки запуска приводов кранов на открытие.



## Лицензия

MIT

[http://opensource.org/licenses/MIT](http://opensource.org/licenses/MIT)
