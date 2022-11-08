# **Views**
Views в Masonite — отличный способ вернуть HTML в ваши контроллеры. Views имеют иерархию, 
множество помощников и логических элементов управления, а также отличный способ отделить 
бизнес-логику от логики представления.

## Начало
По умолчанию все шаблоны находятся в каталоге `templates`. 
Чтобы создать шаблон, просто создайте файл с расширением `.html`.

```html
<!-- Template in templates/welcome.html -->

<html>
  <body>
    <h1>Hello, {{ name }}</h1>
  </body>
</html>
```
Затем внутри вашего файла контроллера вы можете сослаться на этот шаблон:

```py
from masonite.views import View


class WelcomeController(Controller):

    def show(self, view: View):
        view.render('welcome', {"name": "Joe"})
```
Первым аргумент это имя вашего шаблона, а вторым аргументом должен быть словарь, 
на ключи которого вы ссылаетесь внутри своего шаблона.

```py
from masonite.views import View

class WelcomeController(Controller):

# Template is templates/greetings/welcome.html
def show(self, view: View):
    view.render('greeting.welcome', {"name": "Joe"})
```

##
## Общий доступ ко view
Совместное использование view — это когда вы хотите, чтобы переменная была доступна во всех шаблонах. 
Это может быть аутентифицированный пользователь, год авторского права или 
что-то еще.

В метод `share` класса `View`, необходимо добавить словарь с данными:
```py
from masonite.facades import View

View.share({'copyright': '2021'})
```

Затем в шаблоне сделать так:
```html
<div>
    Copyright {{ copyright }}
</div>
```

Обычно вы будете делать это в своем Service Provider (если у вас его нет, вы должны его создать):
```py
from masonite.facades import View


class AppProvider(Provider):

    def register(self):
        View.share({'copyright': '2021'})
```
##
## Составление представлений (View Composing)
Подобно совместному использованию view, view composing позволяет вам обмениваться данными между 
определенными шаблонами.
```py
View.composer('dashboard', {'copyright': '2021'})
```

Вы также можете передать список шаблонов:
```py
View.composer(['dashboard', 'dashboard/users'], {'copyright': '2021'})
```

Обычно вы будете делать это в своем Service Provider (если у вас его нет, нужно создать):
```py
from masonite.facades import View


class AppProvider(Provider):
    def register(self):
        View.composer('dashboard', {'copyright': '2021'})
```
## Помощники (Helpers)
Во view довольно много встроенных помощников. Вот список всех помощников.

### Request
Вы можете получить данные из `request`:
```html
<p> Path: {{ request().path }} </p>
```
### Asset (статические файлы)
Получить местоположение статических файлов.

Вы можете создать путь к ресурсу с помощью `asset`:
```html
...
<img src="{{ asset('s3', 'profile.jpg') }}" alt="profile">
...
```
В результате URL-адрес отобразится следующим образом:
```html
<img src="https://s3.us-east-2.amazonaws.com/bucket/profile.jpg" alt="profile">
```
См. конфигурацию `filesystems.py`, чтобы узнать, как настроить пути.

### Поле CSRF
Вы можете создать скрытое поле CSRF токена для использования с формами:
```html
<form action="/some/url" method="POST">
    {{ csrf_field }}
    <input ..>
</form>
```
### CSRF-токен
Полезно для интерфейсов JS где вам нужно передать CSRF токен серверу для 
вызова AJAX.
```html
<p> Token: {{ csrf_token }} </p>
```
### Текущий пользователь
Вы также можете получить текущего аутентифицированного пользователя. На подобии `request.user()`.
```html
<p> User: {{ auth().email }} </p>
```
### Маршрут (Route)
Получить маршрут по его имени, используя метод `route`:
```html
<form action="{{ route('route.name') }}" method="POST">
    ..
</form>
```
Если ваш маршрут содержит переменные, которые вам нужно передать, вы можете указать словарь в 
качестве второго аргумента.
```html
<form action="{{ route('route.name', {'id': 1}) }}" method="POST">
  ..
</form>
```
или список:
```html
<form action="{{ route('route.name', [1]) }}" method="POST">
  ..
</form>
```
### Back (вернуться назад)
Это полезно для перенаправления на предыдущую страницу. Метод `request.back()` перенаправит к этой 
конечной точке. Это полезно использовать, когда нужно вернуться на страницу после отправки формы с 
ошибками:
```html
<form action="/some/url" method="POST">
  {{ back(request().path) }}
</form>
```
Теперь, когда форма отправлена, и вы хотите отправить пользователя обратно, тогда в вашем контроллере 
нужно сделать:
```py
def show(self, response: Response):
    # Some failed validation
    return response.back()
```
### Сессия
Вы можете получить доступ к сессии:
```html
<p> Error: {{ session().get('error') }} </p>
```
### Конфигурация
Чтобы получить значения конфигурации в ваших шаблонах:
```html
<h2> App Name: {{ config('application.name') }}</h2>
```
### Куки
Получаем куки:
```html
<h2> Token: {{ cookie('token') }}</h2>
```
### URL
Получить URL-адрес:
```html
<form action="{{ url('/about', full=True) }}" method="POST">
  
</form>
```
### DD (debug)
При отладке, вы можете использовать `dd()`:
```html
{{ dd(variable) }}
```
##
## Фильтры
Jinja2 позволяет добавлять фильтры к вашим views. Прежде чем мы объясним, как добавлять фильтры ко 
всем вашим шаблонам, давайте объясним, что такое фильтр views.

Фильтры могут быть прикреплены к переменным view, чтобы изменять и модифицировать их. Например, 
у вас может быть переменная, которую вы хотите превратить в слаг:
```html
{{ variable|slug }}
```

В Masonite этот фильтр слагов представляет собой просто функцию, которая принимает переменную в 
качестве аргумента и выглядит как простая функция, подобная этой:
```py
def slug(variable):
  return variable.replace(' ', '-')
```
Вот и все! Важно отметить, что переменная которую фильтруем, всегда передается в качестве 
первого аргумента, а все остальные параметры передаются после, поэтому мы могли бы сделать что-то вроде:
```html
{{ variable|slug('-') }}
```
и тогда наша функция будет выглядеть так:
```py
def slug(variable, replace_with):
  return variable.replace(' ', replace_with)
```
### Добавление фильтров
Добавление фильтров обычно выполняется внутри вашего Service Provider:
```py
from masonite.facades import View


class AppProvider(Provider):

    def register(self):
        View('slug', self.slug)

    @staticmethod
    def slug(item):
        return item.replace(' ', '-')
```
## Тесты (View Tests)
Тесты view — это настраиваемые логические выражения, которые можно использовать в ваших шаблонах. 
Мы можем запустить логические тесты для определенных объектов, чтобы подтвердить, что они проходят 
тест. Например, мы можем проверить, является ли пользователь владельцем такой компании:
```html
<div>
    <div data-gb-custom-block data-tag="if">
        hey boss
    </div>
  
    <div data-gb-custom-block data-tag="else">
        you are an employee
    </div>
</div>
```
Для этого нам нужно добавить тест на класс `View`. Мы можем сделать это внутри Service Providers.

Код прост и выглядит примерно так:
```py
from masonite.facades import View


def a_company_owner(user):
    # Returns True or False
    return user.owner == 1


class AppProvider(Provider):

    def register(self):
        # template alias
        View.test('a_company_owner', a_company_owner)
```
Вот и все! Теперь мы можем использовать `a_company_owner` в наших шаблонах точно так же, как в 
первом фрагменте кода выше!

!!! note 
    Обратите внимание, что мы предоставили только функцию и ничего не создавали. Функция или объект, 
    должны иметь один параметр (объект или строка) который мы тестируем.
##
## Добавление Environments
Environments во views — это каталоги шаблонов. Если вы разрабатываете пакеты или создаете модульное 
приложение, вам может потребоваться зарегистрировать разные каталоги для шаблонов. Это позволит 
Masonite находить ваши шаблоны для рендеринга. 
**Лучше всего это сделать в методе регистрации service provider.**

Есть 2 отдельных типа загрузчиков.

Первый загрузчик является "пакетным". Он используется при регистрации пакета. Для этого вы должны 
зарегистрировать его с помощью пути к модулю пакета. Это будет работать для большинства каталогов, 
которые являются пакетами.
```py
from masonite.facades import View

View.add('module_name/directory')
```

Второй загрузчик `FileSystem`. Его следует использовать, когда путь к каталогу **НЕ** является модулем, 
а просто путем к файловой системе:
```py
import os
from jinja2.loaders import FileSystemLoader

from masonite.facades import View

View.add(
    os.path.join(os.getcwd(), 'package_views'), 
    loader=FileSystemLoader
)
```
## Синтаксис представлений (View)
### Jinja2
Ниже приведены некоторые примеры синтаксиса Jinja2, который Masonite использует для построения 
шаблонов.

#### Строковые операторы
Важно отметить, что операторы Jinja2 могут быть переписаны с помощью строковых операторов, 
а строковые операторы **предпочтительнее** в Masonite. По сравнению со строковыми операторами Jinja2, 
оценивается вся строка.

Итак, синтаксис Jinja2 выглядит так:
```html
<div data-gb-custom-block data-tag="if">
  <p>do something</p>
</div>
```
Это можно переписать следующим образом с синтаксисом строкового оператора:
```html
@if expression
  <p>do something</p>
@endif
```
Однако важно отметить, что ничего другого не должно быть в строке при выполнении этих действий. 
Например, вы НЕ МОЖЕТЕ сделать так:
```html
<form action="@if expression: 'something' @endif">

</form>
```
Но вы можете добиться этого с помощью обычного форматирования:
```html
<form action="
  <div data-gb-custom-block data-tag="if"> 'something' </div>">
</form>
```
Какой синтаксис выбрать, решать вам.

!!! note warning
    Обратите внимание, что если вы используете символ `@`, который не должен отображаться с помощью 
    Masonite, это приведет к ошибке. Например, когда вы используете `@media` теги в CSS. В этом 
    случае вам нужно будет обернуть этот оператор внутри двойных <code>**``**</code>.

#### Переменные
Вы можете отобразить переменный или строковый текст, используя `{{ }}`:
```html
<p>
  {{ variable }}
</p>

<p>
  {{ 'hello world' }}
</p>
```
#### Оператор If
Оператор `if` похож на python, но требуют `endif`.

Строковые операторы:
```html
@if expression
    <p>do something</p>
@elif expression
    <p>do something else</p>
@else
    <p>above all are false</p>
@endif
```
Использование альтернативного синтаксиса Jinja2:
```html
<div data-gb-custom-block data-tag="if"></div>
  <p>do something</p>

<div data-gb-custom-block data-tag="elif"></div>
  <p>do something else</p>

<div data-gb-custom-block data-tag="else">
  <p>above all are false</p>
</div>
```
#### Цикл For
Цикл `for` похож на обычный синтаксис python.

Строковые операторы:
```html
@for item in items
  <p>{{ item }}</p>
@endfor
```
Использование альтернативного синтаксиса Jinja2:
```html
<div data-gb-custom-block data-tag="for">
  <p>{{ item }}</p>
</div>
```
#### Include
Оператор `include` полезен для включения других шаблонов.

Строковый оператор:
```html
@include 'components/errors.html'

<form action="/">
  
</form>
```
Использование альтернативного синтаксиса Jinja2:
```html
<div data-gb-custom-block data-tag="include" data-0='components/errors.html'></div>

<form action="/">

</form>
```

В любом месте, где у вас есть повторяющийся код, вы можете поместить его в `include`. 
Эти шаблоны будут иметь доступ ко всем переменным в текущем шаблоне.

#### Расширение (Extends)
Это полезно, когда дочерний шаблон расширяет родительский шаблон. В каждом шаблоне может быть 
только одно расширение:

Строковые операторы:
```html
@extends 'components/base.html'

@block content
  <p> read below to find out what a block is </p>
@endblock
```
Использование альтернативного синтаксиса Jinja2:
```html
<div data-gb-custom-block data-tag="extends" data-0='components/base.html'></div>

<div data-gb-custom-block data-tag="block">
  <p> read below to find out what a block is </p>
</div>
```
#### Блоки (Blocks)
Блоки — это разделы кода, которые можно использовать в качестве заполнителей для родительского шаблона. 
Они полезны только при использовании с `extends`. Шаблон `«base.html»` является родительским шаблоном 
и содержит блоки, которые определены в дочернем шаблоне `«blocks.html»`.

Строковые операторы:
```html
<!-- components/base.html -->

<html>
  <head>
    @block css
      <!-- здесь будет вставлен блок с именем "css", определенный в дочернем шаблоне -->
    @endblock
  </head>
  <body>
    
    <div class="container">
      @block content
      <!-- здесь будет вставлен блок с именем "content", определенный в дочернем шаблоне -->
      @endblock
    </div>

    @block js
    <!-- здесь будет вставлен блок с именем "js", определенный в дочернем шаблоне -->
    @endblock

  </body>
</html>
```
```html
<!-- components/blocks.html -->

@extends 'components/base.html'

@block css
  <link rel=".." ..>
@endblock

@block content
  <p> This is content </p>
@endblock

@block js
  <script src=".." />
@endblock
```

Использование альтернативного синтаксиса Jinja2:
```html
<!-- components/base.html -->

<html>
  <head>
    <div data-gb-custom-block data-tag="block">
      <!-- здесь будет вставлен блок с именем "css", определенный в дочернем шаблоне -->
    </div>
  </head>
  <body>
  
    <div class="container">
      <div data-gb-custom-block data-tag="block">
        <!-- здесь будет вставлен блок с именем "содержимое", определенный в дочернем шаблоне -->
      </div>
    </div>

    <div data-gb-custom-block data-tag="block">
      <!-- здесь будет вставлен блок с именем "js", определенный в дочернем шаблоне -->
    </div>
  
  </body>
</html>
```
```html
<!-- components/blocks.html -->

<div data-gb-custom-block data-tag="extends" data-0='components/base.html'></div>

<div data-gb-custom-block data-tag="block">
  <link rel=".." ..>
</div>

<div data-gb-custom-block data-tag="block">
  <p> This is content </p>
</div>

<div data-gb-custom-block data-tag="block">
  <script src=".." />
</div>
```

Как видите, блоки являются фундаментальными и могут быть определены с помощью операторов Jinja2 и line. 
Это позволяет вам структурировать ваши шаблоны и иметь меньше повторяющегося кода.
