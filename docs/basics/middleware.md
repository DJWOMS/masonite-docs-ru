# Middleware
Middleware (промежуточное программное обеспечение) является чрезвычайно важным аспектом 
веб-приложений, поскольку оно позволяет запускать важный код до или после каждого 
запроса или определенных маршрутов. 
В этой документации мы поговорим о том, как работает middleware, 
как использовать и создать свои собственные middleware.
## Начало
Классы middleware размещаются внутри класса `Kernel`. 
Все middleware — это просто классы, содержащие методы `before` и `after`.

Всего существует четыре типа middleware:

- Middleware запускается перед каждым запросом.
- Middleware запускается после каждого запроса.
- Middleware запускается перед определенными маршрутами.
- Middleware запускается по определенным маршрутам.

## Конфигурация
У нас есть один из двух атрибутов конфигурации, с которыми нам нужно работать. 
Эти атрибуты находятся в нашем файле `Kernel.py`, `http_middleware` и `route_middleware`.

`http_middleware` это простой список, который должен содержать ваши классы middleware. 
Этот атрибут представляет собой список, потому что все middleware запускается 
последовательно одно за другим, аналогично middleware Django.

В файле `Kernel.py` этот тип middleware может выглядеть примерно так:
```py
from masonite.middleware import EncryptCookies

class Kernel:

    http_middleware = [
        EncryptCookies
    ]
```

Middleware будет запускаться при каждом входящем запросе к приложению независимо от того, 
был найден маршрут или нет.

## Middleware маршрутизации
Middleware маршрута это словарь с произвольным именем в качестве ключа и списком middleware. 
Это сделано для того, чтобы мы могли указать middleware на основе ключа в нашем файле маршрутов.

В файле `Kernel.py` это может выглядеть примерно так:
```py
from app.middleware.RouteMiddleware import RouteMiddleware

class Kernel:

    #..
    
    route_middleware = {
        "web": [
            SessionMiddleware,
            HashIDMiddleware,
            VerifyCsrfToken,
        ],
    }
```
По умолчанию все маршруты внутри файла `web.py` будут забирать список по ключу `web`.

## Параметры middleware
Вы можете передавать параметры из ваших маршрутов в middleware в тех случаях, когда middleware 
должно работать по-разному в зависимости от вашего маршрута.

Сделать это можно с помощью двоеточия `:` рядом с именем middleware вашего маршрута, 
а затем передать эти параметры методам middleware `before` и `after`.

Например, мы можем создавать middleware для регулирования запросов, и в наших маршрутах у нас 
есть что-то вроде этого:

```py
Get('/feeds', 'FeedController').middleware('throttle:2,100')
```

Обратите внимание на синтаксис `throttle:2,100`. 2 и 100 будут переданы в методы `before` и 
`after` вашего middleware:
```py
class ThrottleMiddleware:

    def before(self, request, response, minutes, requests):
        # throttle requests

    def after(self, request, response, minutes, requests):
        # throttle requests
```

## Параметры запроса
Подобно тому, как мы передаём значения middleware с помощью двоеточия `:`, 
также можно использовать `@` для передачи значения параметра.

Например, мы можем создать маршрут и middleware, подобные этому.
```py
Get('/dashboard/@user_id/settings', 'FeedController').middleware('permission:@user_id')
```
Если перейти по адресу, `/dashboard/152/settings`, то значение **152** будет передано 
в методы middleware `before` и `after`.

## Создание middleware
Middleware:

- Может находиться где угодно в вашем проекте.
- Наследуются от базового класса `Middleware`.
- Должен иметь методы `before` и `after`, которые принимают параметры запроса и ответа.
```py
from masonite.middleware import Middleware

class AuthenticationMiddleware(Middleware):
    """ Middleware class
    """

    def before(self, request, response):
        #..
        return request

    def after(self, request, response):
        #..
        return request
```

!!! note warning
    Важно отметить, что для продолжения жизненного цикла запроса необходимо вернуть 
    класс запроса (Request). Если вы не вернете класс запроса, никакой другой middleware не 
    будет запускаться после этого middleware.

Вот и все! Если бы мы хотели, чтобы это выполнялось после запроса, мы могли бы 
использовать точно такую же логику в методе `after`.

## Использование middleware
Если мы используем middleware для маршрута, нам нужно указать, какой маршрут должен выполнять 
middleware. Просто добавите метод `.middleware()` к маршрутам. Это будет выглядеть примерно так:
```py
Route.get('/dashboard', 'DashboardController@show').name('dashboard').middleware('auth')
Route.get('/login', 'LoginController@show').name('login')
```

