# Таблица связей через функцию

## Описание
Известно, что для правильной работы модели данных, таблицы должны быть соединены между собой только 1 полем, а что делать если у нас в 2 или более таблицах от 2 одинаковых полей?
Мы же не хотим держать SSyn-таблицы внутри модели? Иначе у нас будут не верные расчёты..

![image](https://user-images.githubusercontent.com/8188055/189489304-47246296-1770-4478-ae8d-c67344823c6f.png)

Тут нам на помощь приходит функция создания промежуточной таблицы связей между таблицами.
Помните, что функции должны быть написаны в скрипте ранее, чем они будут вызваны.
И так, синтаксис:
```
CALL ls_LinkTable ('имя новой или существующей таблицы связей', 'имя таблицы из которой нужно забрать поля', 'поля перечисленные через запятую' );
```
Например, написав 2 строчки в коде, мы сделаем связи между 2 таблицами по 3 полям:
```
CALL ls_LinkTable ('plan-fact_link', 'table_sales', 'period, id_goods, id_shop');
CALL ls_LinkTable ('plan-fact_link', 'table_plan', 'period, id_goods, id_shop');
```

![image](https://user-images.githubusercontent.com/8188055/189489443-57ebaaad-177d-42f8-8f57-5dc24f10ea3a.png)

Результатом выполнения функций - будет новая таблица, в которой будет ключевое поле и 3 поля из 2 таблиц, в основных таблицах останется только ключевое поле

**Важно!** На ключевое поле - используется функция AUTONUMBER() для оптимизации работы Qlik Engine, и уменьшения объемов данных.

![image](https://user-images.githubusercontent.com/8188055/189489593-383f5b8b-8b30-41a1-b86c-6117adb193c7.png)

Тестовое приложение можно забрать тут: [link tables.zip](https://github.com/bintocher/qlik-lib/files/9540704/link.tables.zip)

## Код функции
```
SUB ls_LinkTable (ls_linkTableName, ls_table, ls_fields)

// Создание или обновление таблицы связей.
//
// Поля, перечисленные в третьем параметре будут удалены из исходной таблицы, указанной во втором параметре и помещены в таблицу связей, указанной в первом параметре.
// Если таблица связей не существует, то она будет создана. Если таблица связей существует, то она будет обновлена
//
// @параметр1 String: Название новой или существующей таблицы связей.
// @параметр2 String: Название таблицы, из которой будут загружены поля.
// @параметр3 String: Названия полей, разделенных запятой которые будут помещены в таблицу связей.
//
// @синтаксис: CALL ls_LinkTable('LinkTableName', 'SourceTableName', 'Field1, Field2, ...');
// @синтаксис: CALL ls_LinkTable('LinkTableName', 'SourceTableName2', 'Field1, Field2, ...');

// Генерируем произвольное наименование временной таблицы
LET ls_LinkTableTemp = '$(ls_linkTableName)' & '_temp_' & KEEPCHAR(NOW(),'0123456789');

[$(ls_LinkTableTemp)]:
NOCONCATENATE LOAD DISTINCT
	$(ls_fields),
	AutoNumberHash128('$(ls_fields)',$(ls_fields)) as [%$(ls_linkTableName)_Key]
RESIDENT $(ls_table);

LEFT JOIN ($(ls_table))	// Join key from link table to source table
LOAD [%$(ls_linkTableName)_Key], $(ls_fields) RESIDENT [$(ls_LinkTableTemp)];

DROP FIELDS $(ls_fields) FROM $(ls_table);

IF NOT ISNULL(TableNumber('$(ls_linkTableName)')) THEN
	CONCATENATE ([$(ls_linkTableName)])
	LOAD * RESIDENT [$(ls_LinkTableTemp)];
	DROP TABLE [$(ls_LinkTableTemp)];
ELSE
	RENAME TABLE [$(ls_LinkTableTemp)] TO [$(ls_linkTableName)];
ENDIF

SET ls_LinkTableTemp=;

// Применяем AUTONUMBER для сокращения объема данных по нашему ключевому полю
AUTONUMBER [%$(ls_linkTableName)_Key] USING '$(ls_linkTableName)';

END SUB;
```


## Код генерации тестовых данных

Код должен быть расположен после написания кода функции
```
LET varPeriodStart = NUM(Today() - 60);

table_sales:
LOAD
    DATE(FLOOR($(varPeriodStart) + (rand() * 60))) AS period
    , floor(rand()*150)+1 AS id_goods
    , floor(rand()*10)+1 AS id_shop
    , floor(rand() * 1000) + 10 AS fact_sales
    , floor(rand() * 20) + 1 AS fact_quant
AUTOGENERATE 10000
;


table_plan:
LOAD
    DATE(FLOOR($(varPeriodStart) + (rand() * 60))) AS period
    , floor(rand()*150)+1 AS id_goods
    , floor(rand()*10)+1 AS id_shop
    , floor(rand() * 1000) + 10 AS plan_sales
    , floor(rand() * 20) + 1 AS plan_quant
AUTOGENERATE 10000
;


table_goods:
LOAD
    FIELDVALUE ('id_goods', RowNo()) as id_goods
    , 'good_' & FIELDVALUE ('id_goods', RowNo()) as goods_name
AUTOGENERATE FIELDVALUECOUNT ('id_goods');

table_shops:
LOAD
    FIELDVALUE ('id_shop', RowNo()) as id_shop
    , 'shop_' & FIELDVALUE ('id_shop', RowNo()) as shop_name
AUTOGENERATE FIELDVALUECOUNT ('id_shop');


CALL ls_LinkTable ('plan-fact_link', 'table_sales', 'period, id_goods, id_shop');
CALL ls_LinkTable ('plan-fact_link', 'table_plan', 'period, id_goods, id_shop');

```
