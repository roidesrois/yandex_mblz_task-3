# Задание 3

Мобилизация.Гифки – сервис для поиска гифок в перерывах между занятиями.

Сервис написан с использованием [bem-components](https://ru.bem.info/platform/libs/bem-components/5.0.0/).

Работа избранного в оффлайне реализована с помощью технологии [Service Worker](https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API/Using_Service_Workers).

Для поиска изображений используется [API сервиса Giphy](https://github.com/Giphy/GiphyAPI).

В браузерах, не поддерживающих сервис-воркеры, приложение так же должно корректно работать, 
за исключением возможности работы в оффлайне.

## Структура проекта

  * `gifs.html` – точка входа
  * `assets` – статические файлы проекта
  * `vendor` –  статические файлы внешних библиотек
  * `service-worker.js` – скрипт сервис-воркера

Открывать `gifs.html` нужно с помощью локального веб-сервера – не как файл. 
Это можно сделать с помощью встроенного в WebStorm/Idea веб-сервера, с помощью простого сервера
из состава PHP или Python. Можно воспользоваться и любым другим способом.



##  Bug fixing

1.  Неверное местоположение файла service-worker.js, так область видимости service-worker-а ограничивается папкой /assets. Файл перенесен в 
	корневую директорию `entrance-task-3`. Теперь scope охватывает все приложение, то есть будут обрабатываться все запросы. Также был изменён путь до файла `service-worker.js` в blocks.js и путь до kv-keeper.js


2.  Отсутствие слежения за файлом gifs.html. Решение: в функцию `needStoreForOffline()` добавлена проверка на `gifs.html`.

    cacheKey.includes('gifs.html')

3.  Некорректная обработка события fetch. Необходимо чтобы приложение сперва пыталось скачивать файлы, и только в случае ошибки, 
	запрашивала их из кеша. Исправлено следующим образом:

	let response;
	if (needStoreForOffline(cacheKey)) {
	    response = fetchAndPutToCache(cacheKey, event.request);
	} else {
	    response = fetchWithFallbackToCache(event.request);
	}


4. Невозможно обновить статику из директорий vendor и assets. 
   Для того, чтобы приложение корректно обновляло статику нужно изменить `CACHE_VERSION` тогда проверка в функции `deleteObsoleteCaches` будет корректно срабатывать.

   const CACHE_VERSION = '1.0.0-fixed';

5. Возможность переключения в оффлайн-режим после первого запроса

   Решение: добавлено предварительное кэширование всех ресурсов приложения на стадии установки 'servis-worker'-a. Теперь переключается в оффлайн-режим сразу же после первого запроса.




## Ответы на вопросы, закомментированные в `service-worker.js`:

Вопрос №1. Зачем нужен этот вызов метода `skipWaiting()`?

	Обновленный `service-worker` не активируется, пока загружаются страницы, 
	использующие старый `service-worker`. Вызов skipWaiting() немедленно активирует
	работу нового service-worker'a, без ожидания


Вопрос №2. Зачем нужен этот вызов метода `clients.claim()`

    Вызов метода client.claim() активирует `service-worker` на все страницы, входящие в его область видимости без необходимости перезагрузки страницы, позволяет ему немедленно начать перехват запросов.


Вопрос №3. Для всех ли случаев подойдёт такое построение ключа?

	const cacheKey = url.origin + url.pathname;

   	Не подойдет для GET-запросов с использованием параметра поиска


Вопрос №4. Зачем нужна эта цепочка вызовов? `name !== CACHE_VERSION`.

  	Для очистка ресурсов, использованных в предыдущей версии скрипта сервис-воркера.
	Метод filter определяет изменилось ли название версии 'servic-worker'-a, если да, то этот кэш необходимо очистить.
	Метод map вызывает функцию для каждого элемента массива. 


Вопрос №5. Для чего нужно клонирование?

	Потоки запроса и ответа могут быть прочитаны только единожды. Чтобы ответ был получен браузером и сохранен в кеше — нам нужно клонировать его. Так, оригинальный объект отправится браузеру, а клон будет закеширован. Оба они будут прочитаны единожды.




