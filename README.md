На днях появилась задача разобраться в проблеме обмена данными по протоколу XML-RPC между Drupal сайтами.

Для понимания проблемы, введем терминологию.
* Клиент - Drupal сайт, запрашивающий информацию с Сервера
* Сервер - Drupal сайт, отвечает на запросы с Клиента

Клиент отправляет запрос на Сервер по протоколу XML-RPC
Сервер обрабатывает запрос и отправляет Клиенту ответ
Клиент получает данные, не может выполнить обработку и выдает ошибку:
```sh
[http://example.ru/xmlrpc.php] Parse error for system.getCapabilities on http://example.ru/xmlrpc.php: ﻿ xmlrpc specUrlhttp://www.xmlrpc.com/spec specVersion1 faults_interop specUrlhttp://xmlrpc-epi.sourceforge.net/specs/rfc.fault_codes.php specVersion20010516 system.multicall specUrlhttp://www.xmlrpc.com/discuss/msgReader$1208 specVersion1 introspection specUrlhttp://scripts.incutio.com/xmlrpc/introspection.html specVersion1. Failed to connect to the URL example.ru.
```
То есть парсер на стороне Клиента не смог обработать данные, так как они не соответствуют спецификации.

Первым делом надо проверяю требования и получаемые данные.
Обнаружил, что сервер в каждом ответе в начале шлёт спец.символ BOM.

Причина кроется в файловом редакторе. Вероятно программист правил на сайте файл php и сохранил его в Блокноте (операционная система Windows).

##### Поиск и удаление BOM в файлах сайта
#
Поиск файлов:
```sh
find . -name "*.php" | xargs grep -l $'\xEF\xBB\xBF'
find . -name "*.inc" | xargs grep -l $'\xEF\xBB\xBF'
grep -rnw --include=*.{module,inc,php} ./ -e $'\xEF\xBB\xBF'
```
Открыть найденные файлы в редакторе Vim:
```sh
vi filename.php
```
Найти в файле спец.символ:
```sh
/fe или /EF
```
Удалить и установить режим сохранения без bom:
```sh
:set nobomb
```
Сохранить файл и выйти
```sh
:wq
```
Выполнить пункты с другими найденными файлами.

По завершению повторить обмен между Клиентом и Сервером.

Успехов в поиске.
