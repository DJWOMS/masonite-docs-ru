# Маршрутизация masonite (routes)

Masonite обладает действительно мощным механизмом маршрутизации. 
Маршрутизация помогает связать URL-адрес с контроллером.

## Создание маршрута
Маршруты создаются путем импорта класса Route и определением метода HTTP, который вы хотели бы 
использовать с URL-адресом и контроллером. Эти маршруты должны находиться в списке ROUTES внутри вашего файла маршрутов.
``` py
from masonite.routes import Route

ROUTES = [
    Route.get('/welcome', 'WelcomeController@show')
]
``` 
 
Первый параметр — это URL-адрес, который должен быть доступен в вашем приложении. 
В приведенном выше примере это позволит любому пользователю перейти по URL-адресу `/welcome`.
Второй параметр — это контроллер, к которому вы хотите привязать этот маршрут.



### Доступные методы
Вы можете определить любой из доступных методов:

``` py 
Route.get('/welcome', 'WelcomeController@show')
Route.post('/welcome', 'WelcomeController@show')
Route.put('/welcome', 'WelcomeController@show')
Route.patch('/welcome', 'WelcomeController@show')
Route.delete('/welcome', 'WelcomeController@show')
Route.options('/welcome', 'WelcomeController@show')
Route.view('/url', 'view.name', {'key': 'value'})
Route.resource('/users', 'UsersController')
Route.api('/users', 'UsersApiController')
``` 
 
В дополнение к этим методам вы можете использовать встроенные маршруты:
 
``` py
Route.redirect('/old', '/new', status=301)
Route.permanent_redirect('/old', '/new')
```

### Привязка контроллера
Есть несколько способов привязать контроллер к маршруту.
 
#### Привязка к строке
Вы можете использовать привязку строки, определяющую класс контроллера и его метод `{ControllerClass}@{controller_method}`:

``` py 
Route.get('/welcome', 'WelcomeController@show')
``` 

!!! note info
    При использовании привязки строк необходимо убедиться, что этот класс контроллера может быть правильно 
    импортирован и что класс контроллера находится зарегистрированном местоположении контроллера.
 
Обратите внимание, что это предпочтительный способ, поскольку он позволяет избежать циклических импортов, 
поскольку в файле маршрута не требуется импорт.
 

#### Привязка класса
Вы можете импортировать свои контроллеры в файл маршрута и указать имя класса или метода:
``` py  
from app.controllers import WelcomeController
 
Route.get('/welcome', WelcomeController)
```

Поскольку метод не был определен, метод `__call__` будет привязан к этому маршруту. 
Это означает, что вы должны определить этот метод в своем контроллере:

``` py   
class WelcomeController(Controller):
 
    def __call__(self, request:Request):
        return "Welcome"
``` 
 
Для удобства вместо этого вы можете указать метод класса:
``` py  
from app.controllers import WelcomeController
 
Route.get('/welcome', WelcomeController.show)
``` 

#### Привязка экземпляра
 
Вы также можете привязать маршрут к экземпляру контроллера:
``` py  
from app.controllers import WelcomeController
 
controller = WelcomeController()
 
Route.get('/welcome', controller)
```

Поскольку метод не был определен, метод `__call__` будет привязан к этому маршруту. Это означает, 
что вы должны определить этот метод в своем контроллере:
``` py  
class WelcomeController(Controller):
 
    def __call__(self, request:Request):
        return "Welcome"
```
---
## Варианты маршрута
 
Вы можете определить несколько доступных методов на ваших маршрутах, чтобы изменить их поведение во время запроса.
 
### Middlewares
Вы можете добавить одно или несколько middlewares в Routes:
``` py  
Route.get('/welcome', 'WelcomeController@show').middleware('web')
Route.get('/settings', 'WelcomeController@settings').middleware('auth', 'web')
``` 
 
Это прикрепит ключ(и) middleware к маршруту, который будет получен из вашей конфигурации middleware позже в запросе.
 
Вы можете исключить одно или несколько middleware для определенного маршрута:
``` py  
Route.get('/about', 'WelcomeController@about').exclude_middleware('auth', 'custom')
``` 

### Имя
Вы также можете указать имя для вашего маршрута. Это используется для получения информации о маршруте в других частях вашего приложения с использованием имени маршрута, которое гораздо более статично, чем URL-адрес.
``` py   
Route.get('/welcome', 'WelcomeController@show').name('welcome')
``` 

### Параметры
Укажите параметры в URL-адресе, которые впоследствии можно будет получить в других частях вашего приложения. 
Сделать это можно, указав имя параметра, прикрепленного к символу `@`
``` py  
Route.get('/dashboard/@user_id', 'WelcomeController@show')
``` 

### Дополнительные параметры
Есть вариант сопоставить маршруты и параметры маршрута. 
Например, сопоставить `/dashboard/user` и `/dashboard/user/settings` с одним и тем же методом контроллера.
В этом случае вы можете использовать необязательные параметры, которые просто заменяют символ `@` на `?`:
``` py  
Route.get('/dashboard/?option', 'WelcomeController@show')
```

### Домен
Укажите поддомен, с которым должен сопоставляться этот маршрут. Например, чтобы маршрут соответствовал только 
поддомену «docs» (docs.example.com):
``` py  
Route.get('/dashboard/@user_id', 'WelcomeController@show').domain('docs')
```

### Типы в routes
Типы маршрутов — это способ сопоставления параметра маршрута по типу. 
Например, чтобы `@user_id` был целым числом. Сделать это можно, добавив после параметра двоеточие `:` и тип:
``` py  
Route.get('/dashboard/@user_id:int', 'WelcomeController@show')
```

#### Доступные типы маршрутов:
* integer
* int (псевдоним для целого числа)
* string
* signed
* uuid
 
### Создание типов маршрутов
Вы также можете создать свои собственные компиляторы маршрутов, если хотите иметь возможность поддерживать 
регулярные выражения для маршрутов.
Все компиляторы маршрутов должны быть добавлены в начало вашего метода `register_routes()` в файле `Kernel.py`.
``` py  
def register_routes(self):
    Route.set_controller_locations(self.application.make("controllers.location"))
    Route.compile("handle", r"([\@\w\-=]+)")
#..
``` 

!!! note "Примечание"
    Методы компиляции должны выполняться до загрузки маршрутов в этот метод, поэтому убедитесь, 
    что он находится вверху. Вы также можете поместить его в любой метод, который стоит перед `register_routes()`.
---
## Группы routes
Группы маршрутов — отличный способ сгруппировать вместе несколько маршрутов с одинаковыми параметрами, 
такими как префикс, или с одним и тем же middleware.
Используйте метод `group()`, который принимает список маршрутов и параметры:
``` py  
ROUTES = [
    Route.group([
        Route.get('/settings', 'DashboardController@settings').name('settings'),
        Route.get('/monitor', 'DashboardController@monitor').name('monitor'),
    ],
    prefix="/dashboard",
    middleware=['web', 'cors'],
    name="dashboard."),
    domain="docs"
]
``` 
 
В приведенном выше примере, имена маршрутов будут `dashboard.settings` и URL-адрес `/dashboard/settings`, 
а у `dashboard.monitor` будет URL-адрес `/dashboard/monitor`.
---
## Route Views
Представления маршрута (route views) — это способ быстро вернуть представление без необходимости создавать контроллер:
``` py  
ROUTES = [
    Route.view("/url", "view.name", {"key": "value"})
]
``` 

При желании вы можете передать http методы:
``` py  
ROUTES = [
    Route.view("/url", "view.name", {"key": "value"}, method=["get", "post"])
]
```
---
## Список маршрутов
Маршруты приложений могут быть перечислены с помощью команды `routes:list`. 
Маршруты будут отображаться в таблице с соответствующей информацией, такой как имя, методы, контроллер и middleware.
 
Маршруты можно фильтровать по методам:

    python craft routes:list -M POST,PUT
 
Фильтрация по названию:

    python craft routes:list -N users
