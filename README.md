# Часть 1. micro frontends.

---
### Disclaimer.

К сожалению не являюсь специалистом по javascript, и тем более по React.  
Запуск приложения после разбивки делать не будем (хотя хотелось бы).

---
### Анализ приложения.
Система представляет собой систему загрузки фото от имени авторизованного пользователя, с возможностью отметить понравившееся фото.
Монолит использует REST API для работы с бэкэндом, в основном CRUD операции.
Пользовательский токен хранится в localStorage браузера. Модель доступа видится в виде упрошенной схемы - аутентифицирован пользователь или нет.


### Бизнес функции.
Пользовательский интерфейс представляет собой SPA приложение, которое на входе отображает форму логина или регистрации. После входа, пользователь попадет на главное
окно приложения, в котором прослеживается блок информации о пользователе и лента загруженных фото с лайкамим.
Функционально можно обозначить такие блоки как:
- Аутентификация пользователя
- Возможность работы с профилем залогиненного пользователя
- Возможность работы с лентой фото, а именно добавить, удалить фото, отметить понравившееся фото или снять отметку.
- Также в интерфейсе и в коде приложения имеются элементы, общие для разных функциональных блоков, например выводы об ошибках, формы ввода или редактирования данных.

Выберем стратегию для проектирования микрофронэндов. Стратегия "автономность команд" будет не подходящей в данном случае, потому что приложение написано на одном стеке технологий и функционал в нем не большой.
Предположу что команд не много (а скорее всего одна) и использовать разные стэки смысла нет.

Вполне логично разделить фронт по функциональным блокам и обернуть каждый блок в отдельный микрофронтенд.  
Рассмотрим "вертикальную нарезку" и "изоляцию". На мой взгляд, это две довольно похожие стратегии и стоит их в определенной степени комбинировать.
С одной стороны четко прослеживаются домены функциональности (аутентификация, пользователь, лента), с другой стороны смущает наличие общих зависимостей в виде единых компонентов системы и вероятная необходимость масштабировать ленту вне зависимости от другой функциональности.
Лента фотографий выглядит как довольно нагруженной, и хорошо бы иметь возможность обновлять данный модуль на новые версии библиотек, независимо от остальных.  
Возможно даже использовать другой фреймворк, более оптимизированный для работы с картинками, но это может усложнить разработку приложения.
Поэтому остановимся на вертикальной нарезке по функциональным доменам.

Итак, со стратегией определились - будем нарезать по следующим функциональным доменам:

Первое, что хотелось бы отметить, это наличие общих для всей системы компонент, как то InfoTooltip и PopupWithForm.  
Идеальным решением, мне кажется, было бы оформить эти компоненты в виде библиотек (одной или несколько), загрузить в npm репозиторий и подключать к микрофронтенду как зависимость.
Это позволит использовать разные версии данных компонент, разрабатывать их в отвязке от основного функционала, не дублировать код, и не делать отдельный микрофронтенд для этого.
В текущей реализации, считаем что у нас пока нет этих библиотек, поэтому будем дублировать компоненты по микрофронтендам, благо их не много.

Новая структура микрофронтенддов находится в [папке microfrontend](frontend/microfrontend).

- `user-auth` - Аутентификация. Отдельные формы для логина и регистрации, которые не зависят от другой функциональности.   
  Результат работы этого модуля - наличие или отсутствие токена пользователя.
  Можно независимо ни от ленты, ни от профиля добавлять разные Identity Provider-ы и инкапсулировать логику аутентификации.
    ```
    user-auth
        └── src
            ├── api
            │		 └── api.js                             // API для REST вызовов логики аутентификации пользователя 
            ├── components  
            │		 ├── Login.js                           // Компонент формы логина пользователя
            │		 └── Register.js                        // Компонент формы регистрации
            └── styles
                └── auth-form                                       // Стили для формы логина
    ```

- `user-profile` - Профиль пользователя. Отдельный блок функциональности, свое место на форме приложения.  
  Зависит от токена пользователя и общих компонент. Дает возможность развивать данную функциональность в параллели с остальной.
    ```
    user-profile
        └── src
            ├── components
            │		 ├── EditAvatarPopup.js                         // Компонент редактирования автара
            │		 └── EditProfilePopup.js                        // Компонент редактирования профиля
            ├── images
            │		 ├── add-icon.svg                               // Иконка кнопки добавления
            │		 └── close.svg                                  // векторная картинка для иконки закрытия popup
            │		 └── edit-icon.svg                              // Иконка кнопки редактирования
            └── styles
                ├── profile                                                 // Стили для отображения блока профиля
                └── popup                                                   // Стили для отображения PopupWithForm
    ```

- `card-store` Лента фото. Функционал просмотра фото. Загрузка, отображение набора, удаление. Также в этот микрофронт будет включен функционал работы с лайками, потому как лайки и их количество атрибут непосредственно объекта карточки фото и на клиенте отображается как единое целое.
  Выглядит потенциально как самый нагруженный компонент так как будет работать с картинками, размер которых может быть довольно большим. Необходимо
  ```
  card-store
          ├── src
          │		 ├── api
          │		 │		 └── api.js                     //  API для работы с бэкендом фото
          │		 ├── components
          │		 │		 ├── AddPlacePopup.js           // Компонент для добавления фото 
          │		 │		 ├── Card.js                    // Компонент для отображения фото в ленте
          │		 │		 └── ImagePopup.js              // Компонент для отображения выбранного фото
          │		 └── images
          │		     ├── close.svg                              // векторная картинка для иконки закрытия popup
          │		     ├── delete-icon.svg                        // векторная картинка для иконки
          │		     ├── like-active.svg                        // векторная картинка для лайка
          │		     └── like-inactive.svg                      // векторная картинка для неактивного лайка
          └── styles
              ├── card                                                    // Стили для карточки фото 
              ├── places                                                  // Стили для формы добавления фото
              └── popup                                                   // Стили для отображения PopupWithForm
  ```

- `main-app` - модуль выступающий в роли контейнера для вышеперечисленных частей.

    ```
    main-app
        ├── index.css                                               // родительский загрузчик стилей
        ├── logo.svg                                                // логотип
        ├── public                                                  // статические артефакты, фавикон, robots.txt и т.п.
        └── src
                 ├── components                                     // React компоненты
                 │		 ├── App.js                             // базовый компонент приложения. Тут должен быть routing и импорт зависимых компонент из других микрофронтендов
                 │		 ├── Footer.js                          // Компонент "подвала". Один на всё приложение, поэтому в main
                 │		 ├── Header.js                          // Компонент "шапки".  Один на всё приложение, поэтому в main
                 │		 ├── InfoTooltip.js                     // Компонент всплывающей информации. Используется несколькими модулями, поэтому в commons
                 │		 ├── Main.js                            // Базовый контейнер для контента. Профиль пользователя и лента фото.
                 │		 └── ProtectedRoute.js                  // Защищенный route. Проверяем аутентифицирован ли пользователь
                 ├── contexts
                 │		 └── CurrentUserContext.js              // Контекст пользователя. Используется всеми модулями. Нет 100% уверенности что он должен быть тут.
                 ├── index.js                                       // Точка входа в приложение.
                 ├── styles                                         // Стили
                 │		 ├── content                            // Стили для контейенра контента Main
                 │		 ├── footer                             // Стили для подвала
                 │		 ├── header                             // Стили для заголовка
                 │		 ├── page                               // Стили для заголовка и подвала.
                 │		 └── popup                              // Стили для заголовка и подвала.
                 └── vendor                                         // Стили для тултипа и формы. 
    ```


В соответствии с функциональными блоками организуем следующую структуру модулей. В каждом модуле, конечно, будут артефакты для сборки приложения, такие как package.json, webpack.config.json и т.п.
Но в данном случае они пропущены, потому как для их написания необходима экспертиза в React и webpack.

###  Инструмент для создания микрофронтендов.

К сожалению, ни в одном из доступных инструментов у меня нет экспертизы, поэтому исходя из вводных, я бы предложил использовать Single SPA.  
При прочих равных с WMF, Single SPA оставит нам возможность для ленты перейти на другой фреймворк, в случае проблем с производительностью.

### Межмодульное взаимодействие.

В монолитной реализации я не увидел каки-либо агрегированных данных. У нас нет тесной связи между модулями. Все модули через REST API обращаются за простыми CRUD операциями в рамках только своей функциональности. Единственный общий артефакт для всех модулей - это токен пользователя, хранящийся в localStorage браузера.
Не считая токена, каких-либо межмодульных взаимодействий реализовывать на текущем этапе не надо.  








