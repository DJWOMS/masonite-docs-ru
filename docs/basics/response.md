# Response
Запрос и ответ (request / response) в Masonite работают вместе. Класс `Response` отвечает за то, 
какие данные возвращаются и как они форматируются. Классы `Request` и `Response` могут содержать 
файлы cookie и заголовки. В большинстве случаев, когда вам нужно получить заголовки или cookie, 
вы должны использовать класс Request, а когда вы устанавливаете файлы cookie и заголовки, 
использовать класс `Response`.

## Cookies
Файлы cookie устанавливаются в классе Response и отображаются как часть заголовков ответа. 
Любые входящие файлы cookie, которые вы, получите из класса Request, можно получить из response:

```py
from masonite.response import Response


def show(self, response: Response):
    response.cookie("key")
```

Вы также можете установить файлы cookie в ответе:
```py
from masonite.response import Response


def show(self, response: Response):
    response.cookie("Accepted-Cookies", "True")
```

Чтобы удалить файлы cookie:
```py
from masonite.response import Response


def show(self, response: Response):
    response.delete_cookie("Accepted-Cookies")
```

## Перенаправление (redirect)
Есть возможность перенаправить пользователя на любой URL.

Перенаправить на URL:
```py
from masonite.response import Response


def show(self, response: Response):
    return response.redirect('/home')
```

Перенаправить туда, откуда пришел пользователь:
```py
from masonite.response import Response


def show(self, response: Response):
    return response.back()
```

Если вы используете метод `back` как часть запроса формы, вам также нужно будет использовать 
`back()` в шаблоне:
```html
<form>
  {{ csrf\_field }}
  {{ back() }}
  <input type="text">
  ...
</form>
```

Перенаправить на маршрут по его имени:
```py
from masonite.response import Response


def show(self, response: Response):
    return response.redirect(name='users.home')
```

Masonite найдет именованный маршрут `users.home`, например:
```py
Route.get('/dashboard/@user_id', 'DashboardController@show').name('users.home')
```

Вы также можете передать параметры маршруту, если этого требует URL:
```py
from masonite.response import Response


def show(self, response: Response):
    return response.redirect(name='users.home', params={"user_id", 1})
```

Наконец, можно передать параметры в URL или маршрут для перенаправления:
```py
def show(self, response: Response):
    return response.redirect(name='users.home', params={"user_id", 1}, query_params={"mode": "preview"})

#== будет перенаправлен на /dashboard/1?mode=preview
```

### with_success
Вы можете сделать перенаправление, запустив сообщение об успешном завершении сеанса:
```py
def show(self, response: Response):
    return response.redirect('/home').with_success("Preferences have been saved!")
```

### with_errors
Перенаправить, запустив сообщение об ошибке или об ошибках в сеанс:
```py
def show(self, request: Request, response: Response):
    return response.redirect('/home').with_errors("This account is disabled!")
```

Вы можете напрямую использовать ошибки проверки:
```py
def show(self, request: Request, response: Response):
    errors = request.validate({
        "name": required
    })
    return response.redirect('/home').with_errors(errors)
```

### with_input
Сделать перенаправление, чтобы повторно заполнить форму при наличии ошибок:
```py
def show(self, request: Request, response: Response):
    errors = request.validate({
        "name": required
    })
    return response.redirect('/home').with_errors(errors).with_input()
```

## Заголовки
Заголовки ответа — это любые заголовки, которые будут найдены в ответе. Masonite позаботится 
обо всех важных заголовках за вас, но бывают случаи, когда вы хотите установить свои собственные 
заголовки.

Большинство входящих заголовков, которые вы хотите получить, можно найти в классе Request, 
но если вам нужно получить какие-либо заголовки в ответе:
```py
from masonite.response import Response


def show(self, response: Response):
    response.header('key')
```

Любые ответы, которые вы хотите вернуть, должны быть установлены в классе Response.

Добавить заголовок в ответ:
```py
from masonite.response import Response


def show(self, response: Response):
    response.header('X-Custom', 'value')
```


## Статус
Установить статус ответа:
```py
from masonite.response import Response

def show(self, response: Response):
    response.status(409)
```

## Загрузка
Вы также можете очень просто загружать изображения, PDF и другие файлы:
```py
def show(self, response: Response):
    return response.download("invoice-2021-01", "path/to/invoice.pdf")
```

Будут добавлены все необходимые заголовки и файл в браузере.

При установке имени, расширение файла будет взято из типа файла. В этом примере будет загружен 
файл как invoice-2021-01.pdf

Если вы хотите принудительно или автоматически загрузить файл в момент  
ответа, вы можете добавить параметр `force` со значением `True`:
```py
def show(self, response: Response):
    return response.download("invoice-2021-01", "path/to/invoice.pdf", force=True)
```
