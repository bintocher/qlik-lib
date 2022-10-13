# Функция для загрузки различных данных из одной таблицы в другую

Подаём на вход 2 параметра:
- название таблицы из которой нужно взять различные значения
- название таблицы в которую нужно поместить различные значения из входной таблицы

Вызов функции:
```
CALL ls_DistinctTable ('my_table_name','my_distinct_table_name');
```

Не забываем удалять изначальную таблицу, при необходимости:

```
DROP TABLE [my_table_name];
```

## Код функции

```
// Функция отдает distinct таблицу обратно
SUB ls_DistinctTable (ls_var_table_in, ls_var_table_out)

    NOCONCATENATE
    [$(ls_var_table_out)]:
    LOAD DISTINCT * RESIDENT [$(ls_var_table_in)];
    ls_var_table_out=;
    ls_var_table_in=;

END SUB;
```
