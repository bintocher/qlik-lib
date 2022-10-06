# Выгрузка данных из таблиц Qlik в SQL

## Введение

На самом деле эта статья не нова, об этом известно было еще в 2019 году

Принципы работы Qlik Sense и QlikView - одинаковы, поэтому **статья применима к обоим продуктам**. Но! Код, написан для Qlik Sense.

Обязательно смотрим раздел **подготовка**

Важные ограничения:

- **Инсерты только текста**. Т.е. вы не сможете положить в БД числа, т.к. при INSERT, числа не должны быть экранированы одинарными кавычками. Я не буду заострять внимание на этот пункт. Пытливые умы смогут это реализовать самостоятельно при необходимости, а я буду ждать pull-реквеста.

- Коннекты к БД, которые я использовал, это - Qlik ODBC Connector Package

## Литература

https://www.w3schools.com/sql/sql_insert.asp

https://support.qlik.com/articles/000064538



## Подготовка

Подготовка состоит из 2 частей:
- создать БД в которую мы будем записывать данные
- разрешить выполнять запросы в QS, которые не возвращают результаты

### Создаем БД

SQL-запрос (подходит как для PostgreSQL так и для MS SQL), который нужно выполнить в бд для создания таблицы с 2 полями **guid** и **personname**, оба с типом **varchar** и **не NULL**
``` sql
CREATE TABLE "Person".qliktable (
	guid varchar NOT NULL,
	personname varchar NOT NULL
);
```

### Выдаем разрешения в Qlik

В зависимости от вашего окружения, нам необходимо отредактировать файл **QvOdbcConnectorPackage.exe.config** который располагается по путям:

Qlik Sense Enterprise
```
C:\Program Files\Common Files\Qlik\Custom Data\QvOdbcConnectorPackage\
```
Qlik Sense Desktop:
```
C:\Users\user-name\AppData\Local\Programs\Common Files\Qlik\Custom Data\QvOdbcConnectorPackage\
```
QlikView:
```
C:\Program Files\Common Files\QlikTech\Custom Data\QvOdbcConnectorPackage\
```

Открываем вышеуказанный файл и редактируем параметр allow-nonselect-queries, ставим значение True. **Обязательно с Большой буквы**

Это изменение, позволяет нам использовать Qlik по-полной, можно выполнять запросы: INSERT/UPDATE/DELETE/DROP


## Функция для бд PostgreSQL
```
SUB ls_ConvertToSQLValues(_temp_table_name,_temp_batch_size)

FOR ls_f=0 to NOOFFIELDS('$(_temp_table_name)')
    [ls_Field_tmp]:
    LOAD
        FIELDNAME($(ls_f),'$(_temp_table_name)') AS [ls_FieldName]
    AUTOGENERATE 1;
NEXT ls_f;

[ls_Concat_Script]:
LOAD
    'SQL INSERT INTO $(_temp_table_name) (' & CONCAT([ls_FieldName],',') & ')' AS [ls_InsertPrefix]
    , 'CHR(39) & ' & CONCAT([ls_FieldName],' & CHR(39) & CHR(44) & CHR(39) & ') & ' & CHR(39)' AS [ls_ValueConcat]
RESIDENT ls_Field_tmp;


LET ls_varInsertPrefix = Peek('ls_InsertPrefix', 0, 'ls_Concat_Script');
LET ls_varValueConcat = Peek('ls_ValueConcat', 0, 'ls_Concat_Script');


DROP TABLES [ls_Field_tmp],[ls_Concat_Script];

[$(_temp_table_name).SQLValues]:
LOAD
    '$(ls_varInsertPrefix)' AS [$(_temp_table_name).Insert]
    , [$(_temp_table_name).Batch_ID] AS [$(_temp_table_name).Batch_ID]
    , CONCAT([$(_temp_table_name).Values], ',' & CHR(10)) AS [$(_temp_table_name).Values]
GROUP BY
    [$(_temp_table_name).Batch_ID];
LOAD
    CEIL ((RecNo()/$(_temp_batch_size))) AS [$(_temp_table_name).Batch_ID]
    // , '$(ls_varValueConcat)' AS [$(_temp_table_name).Values]
    , REPLACE('('&$(ls_varValueConcat)&')', Chr(39)&Chr(39), 'NULL') as [$(_temp_table_name).Values]
RESIDENT [$(_temp_table_name)];

ls_f=;
ls_varInsertPrefix=;
ls_varValueConcat=;

END SUB;
```

## Пример использования для бд PostgreSQL
```
// Указываем имя подключения к БД
LET varDBConnection = 'PostgreSQL_localhost';
// LET varDBConnection = 'Microsoft_SQL_Server_localhost';

// Используем существующее имя в БД,
[postgres."Person".qliktable]:
// но можно и создать/удалить/очистить таблицу из Qlik,
// используя все функции SQL, главное не забывать в конце запроса писать "!EXECUTE_NON_SELECT_QUERY;"
LOAD
    KEEPCHAR(NOW()-RAND(),'0123456789') as guid
	, CEIL(RAND() * 1000) as personname
AUTOGENERATE 5;

LET varLB = CHR(10); // Line Break
// Пишем название таблицы которую будем загружать в БД
LET varTable = 'postgres."Person".qliktable';
// Преобразуем нашу таблицу в понятные для SQL запросы по добавлению данных
CALL ls_ConvertToSQLValues('$(varTable)',10);

FOR i = 0 to NOOFROWS('$(varTable).SQLValues') - 1
    LET varSQL = 'LIB CONNECT TO '& CHR(39) & '$(varDBConnection)' & CHR(39) &'; $(varLB)$(varLB)' &
                Peek('$(varTable).Insert',$(i),'$(varTable).SQLValues') & '$(varLB)' &
                'VALUES $(varLB)' & Peek('$(varTable).Values',$(i),'$(varTable).SQLValues') &
                '$(varLB)$(varLB)!EXECUTE_NON_SELECT_QUERY;$(varLB)$(varLB)DISCONNECT;$(varLB)$(varLB)';
    // Вызвать SQL запрос, добавлен коннект и дисконнект в БД
    $(varSQL)

    varSQL=;
NEXT i;
i=;
// Дроп таблицы
DROP TABLE [$(varTable).SQLValues];
```

## Функция для бд MS SQL
В процессе...
## Пример использования для бд MS SQL
В процессе...
