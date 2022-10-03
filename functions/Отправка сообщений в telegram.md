# Функция отправки сообщений в telegram из Qlik Sense

## Создание бота в telegram

Стучимся к боту https://t.me/botfather и создаем своего бота, получаем токен и запишем его позже в переменную **ls_varTelegramToken**

Необходимо вашего бота добавить в ваш чат/канал

## Как получить ID чата/канала или ID пользователей

Обращаемся к боту : https://t.me/getmyid_bot

Из своего канала/чата пересылаем ему сообщение, и он нам напишет "Forwarded from chat:", вот тут цифры (минус, если есть, не нужно исключать!) - это и будет наш ID-чата для переменной **ls_varTelegramChat_ID**

Для тестирования можно использовать свой ID, и тогда бот будет писать лично вам сообщения, а после тестов уже писать сообщения в чаты/каналы

## Настройка скрипта

Необходимо заполнить переменные: **ls_varTelegramChat_ID** , **ls_varTelegramToken**, **ls_varConnectName**

## Отправка через web-connect

### Настройка web-connect Qlik

Необходимо создать web-коннект и запомнить его имя, позже, в скрипте нужно имя коннекта написать в переменную **ls_varConnectName**

![image](https://user-images.githubusercontent.com/8188055/190596401-11e7e7af-983f-414b-8022-d59a1daee2fd.png)

![image](https://user-images.githubusercontent.com/8188055/190596434-59273cfb-d94b-4af0-aeaf-047cb8c24a4b.png)

![image](https://user-images.githubusercontent.com/8188055/190596578-85e8b67d-a408-4a78-92d1-9ce3c0515a9a.png)

Обязательно в URL указывам в конце слэш "/", иначе получим ошибку

![image](https://user-images.githubusercontent.com/8188055/190596728-2f541f72-c134-447a-82d9-512aa26dd216.png)


### Функция и как с ней работать

Вызов функции и передача в неё сообщения
```
CALL ls_SendTelegramMessage('Привет из Qlik. https://t.me/+wGlxShJJqshlNThi');
```
Код функции, должен быть расположен в скрипте раньше, чем она будет вызвана

```
SUB ls_SendTelegramMessage(ls_varTextMessage)

LET ls_varTelegramChat_ID = '';
LET ls_varTelegramToken = '';
LET ls_varConnectName = '';

[MAP Telegram chars]:
Mapping
LOAD * INLINE [
Неправильный символ, Правильный символ
" " , "%20 "
" H" , "_H"
/ ,"%2f "
\ ,"%5c "
! ,"%21 "
? ,"%3f "
( ,"%28 "
) ,"%29 "
* ,"%2a "
+ ,"%2b "
: ,"%3a "
; ,"%3b "
@ ,"%40 "
# ,"%23 "
$ ,"%24 "
& ,"%26 "
< ,"%3c "
= ,"%3d "
> ,"%3e "
];


LET ls_varTextMessage = trim(KeepChar('$(ls_varTextMessage)', 'абвгдеёжзийклмнопрстуфхцчшщъыьэюяАБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯabcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ.,_- №1234567890%/\!?()*+:;[]@#$&<=>'));
LET ls_varTextMessage = Replace('$(ls_varTextMessage)','[','%5b');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)',']','%5d');
LET ls_varTextMessage = MapSubString('MAP Telegram chars','$(ls_varTextMessage)');
LET ls_varTextMessage = Replace(Replace(Replace('$(ls_varTextMessage)',chr(39),' '),chr(13),' '),chr(10),' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)',chr(10),' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)',chr(13),' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)',chr(39),' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)','  ',' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)','  ',' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)','  ',' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)','	',' ');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)',' .','.');
LET ls_varTextMessage = Replace('$(ls_varTextMessage)',' ,',',');

LET ls_varTextMessage = ApplyCodepage('$(ls_varTextMessage)',65001); //Left(ApplyCodepage('$(ls_varTextMessage)',65001),3000);
LET vUrlFull =  'https://api.telegram.org/bot$(ls_varTelegramToken)/sendMessage?chat_id=$(ls_varTelegramChat_ID)&text=$(ls_varTextMessage)&parse_mode=HTML';

LET ls_varTempTableName = '_temp_' & KEEPCHAR(NOW(),'0123456789')
[$(ls_varTempTableName)]:
LOAD
    NULL() as response
FROM [lib://$(ls_varConnectName)] (url is [$(vUrlFull)],utf8 );

DROP Table [$(ls_varTempTableName)];
LET ls_varTextMessage = ;
LET ls_varTempTableName = ;

END SUB
```

## Отправка через REST API

### Настройка REST-соединения

Создаем новое REST-соединение, все настройки по-умолчанию, а в качестве URL можно использовать https://httpbin.org/get

![image](https://user-images.githubusercontent.com/8188055/193655290-1a1fa028-e7cf-4d4c-9b9a-d316f84c9ad1.png)

Запоминаем название коннекта, в скрипте нужно имя коннекта написать в переменную **ls_varConnectName**

### Функция для отправки через REST API и как с ней работать

Вызов функции и передача в неё сообщения
```
CALL ls_SendTelegramMessage('Привет из Qlik. https://t.me/+wGlxShJJqshlNThi');
```
Код функции, должен быть расположен в скрипте раньше, чем она будет вызвана
```
SUB ls_SendTelegramMessage(ls_varTextMessage)


LET ls_varTelegramChat_ID = '';
LET ls_varTelegramToken = '';
LET ls_varConnectName = '';



LET varUrl = 'https://api.telegram.org/bot$(ls_varTelegramToken)/sendMessage?chat_id=$(ls_varTelegramChat_ID)&text=$(ls_varTextMessage)';  // Перенос строки = %0A

LIB CONNECT TO [$(ls_varConnectName)];

ls_RestConnectTable:
SQL
SELECT
	"__KEY_root"
    , "ok"
FROM JSON (wrap on) "root" PK "__KEY_root"
WITH CONNECTION(
	Url "$(varUrl)"
    );

DROP TABLE ls_RestConnectTable;

END SUB;
```
