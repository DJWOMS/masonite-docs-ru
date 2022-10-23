# Контроллеры masonite (controllers)

Контроллеры — это место, где вы размещаете ответы, которые видите в веб-браузере. 
Ответы могут быть словарями, списками, html (view) или любым классом, который может отображать ответ.
 
## Введение
Вы можете использовать команду `craft` для создания нового базового контроллера или просто создать 
новый контроллер вручную. Контроллеры — это классы с методами, которые сопоставлены с маршрутом.
 
Ваш маршрут может выглядеть примерно так:
``` py
Route.get('/', 'WelcomeController@show')
```
В этом случае маршрут вызовет метод `show` класса `WelcomeController`.
 
Чтобы создать базовый контроллер с помощью команды `craft`, выполните:

    $ python craft controller Welcome
 
Эта команда создаст новый класс контроллера для быстрой настройки. 
Класс контроллера будет выглядеть следующим образом:
``` py
from masonite.controllers import Controller
from masonite.views import View
 
 
class WelcomeController(Controller):
    def show(self, view: View):
        return view.render("")
``` 
 
Вы можете начать создавать свой контроллер и добавлять нужные вам ответы. 

!!! note
    Обратите внимание, что контроллеры наследуют базовый класс `Controller`.
    Это необходимо для того, чтобы Masonite мог найти ваш класс контроллера в маршрутах.

---

## Внедрение зависимости - Dependency Injection
 
В конструктор и в методы контроллера можно передать сервис контейнер (service container) Masonite. 
``` py
def __init__(self, request: Request):
    self.request = request
    
    # ..
    
def show(self, request: Request):
    return request.param('1')
```
 
---

## Ответы - responses
Контроллеры могут вернуть разные типы ответов в зависимости от того, что вам нужно.
 
### JSON
Если нужно вернуть ответ в виде JSON, можете вернуть словарь или список:
``` py
def show(self):
    return {"key": "value"}
``` 
Это будет application/json.
 
### Строки
Вы можете вернуть строки:
``` py
def show(self):
    return "welcome"
``` 
 
### Шаблоны html - view
Если вы хотите вернуть html, используйте метод для рендеринга шаблонов (render):
``` py
def show(self, view: View):
    return view.render("views.welcome")
``` 

### Модели
Есть вариант вернуть модель Masonite ORM напрямую:
``` py
from app.User import User
#..

def show(self, response: Response):
    return User.find(1)
``` 

### Переправление - redirects
Для перенаправление (редиректа), используйте метод `redirect`:
``` py
def show(self, response: Response):
    return response.redirect('/home')
``` 
 
### Другое
Вы можете вернуть любой класс, содержащий метод `get_response()`. Этот метод должен возвращать 
любой из указанных выше типов ответов.
 
## Параметры запроса
Если есть параметр в вашем url, вы можете получить его, указав параметр в методе контроллера:
``` py
Route.get('/users/@user_id', 'UsersController@user')
```
Поскольку параметр `user_id` находится в маршруте, мы можем получить его значение, 
указав параметр в методе контроллера:
``` py
def show(self, user_id):
    return User.find(user_id)
``` 

Другой способ получить параметры маршрута — через класс Request:
``` py
from masonite.request import Request

def show(self, request: Request):
    return User.find(request.param('user_id'))
``` 

---

## Расположение контроллеров
Masonite сможет найти контроллеры (если вы наследуете класс Controller), 
используя привязку строк в месте регистрации контроллеров.
Расположение зарегистрированных контроллеров по умолчанию `app/controllers` определено в файле конфигурации 
проекта `Kernel.py`:
``` py
self.application.bind("controllers.location", "app/controllers")
# ...
Route.set_controller_locations(self.application.make("controllers.location"))
``` 

### Установка местоположений
Вы можете переопределить местоположение зарегистрированных контроллеров в файле `Kernel.py`, 
отредактировав привязку по умолчанию `controllers.location`.
 
### Добавление местоположений
Есть вариант разместить несколько дополнительных контроллеров с помощью `add_controller_locations`:
``` py
from masonite.routes import Route
 
Route.add_controller_locations("app/http/controllers", "other_module/controllers")
``` 
 
Лучшее для этого место — в методе `register_routes()` файла `Kernel.py`.
 
Вы должны сделать это перед регистрацией маршрутов, иначе регистрация маршрутов завершится ошибкой, 
так как Masonite не сможет найти классы контроллеров.
