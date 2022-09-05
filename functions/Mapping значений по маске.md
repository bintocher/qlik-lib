# Маппинг полей с использованием маски с символами * и ?

Необходимо проставить соотствия полям с использованием маски, например значение "1228" превратить в "Колбасы", используя маску "?22?"..

Решение:
```
// Маска значений на замену
wildcard_map:
LOAD * INLINE [
Key, Label
X*, Кондитерские изделия
*99, Кассовая зона
?15*, Колбасы
?17*, Сыры
*, Прочее
]
;

// Создаем через таблицу переменную для функций pick и wildmatch
_MapExpressionTable:
LOAD
'pick(wildMatch($1,' & chr(39)
& concat(Key, chr(39) & ',' & chr(39), FieldIndex('Key', Key))
& chr(39) & '), ' & chr(39)
& concat(Label, chr(39) & ',' & chr(39), FieldIndex('Key', Key))
& chr(39) & ')' as _MapExprression
RESIDENT wildcard_map
;
LET varMapExpr = peek('_MapExprression', -1);
DROP TABLE _MapExpressionTable;
DROP TABLE wildcard_map;

// Использование

Товары:
LOAD *, $(varMapExpr(Код)) as Группа
;
LOAD *  INLINE [
Менеджер, Код
Иванов, X1501
Иванов, C1599
Иванов, A1590
Иванов, A159099
Сидоров, B17809
Сидоров, B17AA6
Сидоров, X99123
Сидоров, L2200
]
;



```
