# Часть 1. micro frontends.

### Анализ
 
Представляет собой систему загрузки фото и проставления рейтинга.
Монолит использует REST API для работы с бэкэндом, в основном CRUD операции.
Пользовательский токен хранится в localStorage браузера. Модель доступа видится в виде упрошенной схемы - аутентифицирован пользователь или нет.

### Бизнес-функции
Видится несколько базовых доменных областей функционала:
- Аутентификация пользователя
  - // TODO : а надо ли ? 
- Профиль пользователя
  - Создание / редактирование пользователя
  - Загрузка аватара
- Каталог картинок
  - Разгрузка / удаление картинки
- Системы рейтинга (лайки)
  - Добавление / удаление лайка
  
Целесообразно разбить монолит на такие же микрофронты по функционалу:  

`user_auth` - модуль отвечающий за аутентификацию пользователя (и возможно в будущем за авторизацию)
Будет получать jwt токен, сохранять, следить за его валидность и предоставлять другим микрофронтам. Хорошо изолируемый от других компонент функционал. 

`user_profile` - модуль отвечающий за профиль пользователя. 
Создание профиля пользователя, его редактирование и удаление при необходимости 

`image_store` - модуль работы с картинками. 
Загрузка картинки в хранилище, получение массива картинок, отображение картинок на клиенте, paging при большом наборе, удаление.

`rating_image` - модуль для работы с лайками фото.

Для объединения модулей (микрофронтов) будем использовать Webpack Module Federation потому, что приложение полностью написано на React, 
есть некоторая связанность компонент, например токен функциональность при работе с токеном должна быть доступна всем компонентам. 
Библиотеки используются одни и те же, с WMF будет удобнее управлять этими зависимостями 
