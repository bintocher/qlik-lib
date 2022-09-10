```
SUB ls_LinkTable (ls_linkTableName, ls_table, ls_fields)

// Создание или обновление таблицы связей.
//
// Поля, перечисленные в третьем параметре будут удалены из исходной таблицы, указанной во втором параметре и помещены в таблицу связей, указанной в первом параметре.
// Если таблица связей не существует, то она будет создана. Если таблица связей существует, то она будет обновлена
//
// @параметр1 String: Название новой или существующей таблицы связей.
// @параметр2 String: Название таблицы из которой будут загружены поля.
// @параметр3 String: Названия полей, разделенных запятой которые будут помещены в таблицу связей.
//
// @синтаксис: CALL ls_LinkTable('LinkTableName', 'SourceTableName', 'Field1, Field2, ...');
// @синтаксис: CALL ls_LinkTable('LinkTableName', 'SourceTableName2', 'Field1, Field2, ...');

// Генерируем произвольное наименование временной таблицы
LET ls_LinkTableTemp = '$(ls_linkTableName)' & '_temp_' & KEEPCHAR(NOW(1),'0123456789');

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
