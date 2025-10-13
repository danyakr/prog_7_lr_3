# Django Tutorial - Полный конспект

## Часть 1: Создание проекта и первого приложения

### Создание проекта
# Создание директории
mkdir djangotutorial

# Создание проекта Django
django-admin startproject mysite djangotutorial
```
Структура проекта:
djangotutorial/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```
Файлы:
- manage.py - утилита командной строки для взаимодействия с проектом
- settings.py - настройки/конфигурация проекта
- urls.py - объявления URL (оглавление сайта)
- asgi.py - точка входа для ASGI-серверов
- wsgi.py - точка входа для WSGI-серверов

### Запуск сервера разработки
```
python manage.py runserver
```

### Создание приложения polls
```
python manage.py startapp polls
```
```
Структура приложения:
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
Разница между проектом и приложением:
- Проект = конфигурация и набор приложений для конкретного сайта
- Приложение = веб-приложение с определенной функциональностью
- Проект может содержать несколько приложений
- Приложение может быть в нескольких проектах

### Первое представление (view)
# polls/views.py
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
### Создание URLconf для приложения
# polls/urls.py (создать новый файл)
```
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```
### Подключение URLconf приложения к проекту
# mysite/urls.py
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```
Функции:
- path() - принимает аргументы: route и view
- include() - позволяет ссылаться на другие URLconf
- При нахождении include(), Django отсекает совпавшую часть URL и отправляет остаток дальше

### Проверка работы
```
python manage.py runserver
```
http://localhost:8000/polls/ в браузере

---

## Часть 2: База данных и модели

### Настройка базы данных
# mysite/settings.py

# Установить часовой пояс
```
TIME_ZONE = 'Europe/Moscow'

# INSTALLED_APPS содержит активированные приложения
INSTALLED_APPS = [
    'django.contrib.admin',      # Административный сайт
    'django.contrib.auth',       # Система аутентификации
    'django.contrib.contenttypes', # Структура типов контента
    'django.contrib.sessions',   # Структура сеанса
    'django.contrib.messages',   # Фреймворк сообщений
    'django.contrib.staticfiles', # Управление статическими файлами
]
```
### Первая миграция
```
python manage.py migrate
```
- Создает таблицы для приложений в INSTALLED_APPS
- По умолчанию использует SQLite

### Создание моделей
# polls/models.py
```
from django.db import models
import datetime
from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")
    
    def __str__(self):
        return self.question_text
    
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
    
    def __str__(self):
        return self.choice_text
```
Типы полей:
- CharField - символьное поле (требует max_length)
- DateTimeField - поле даты и времени
- IntegerField - целочисленное поле
- ForeignKey - связь "многие к одному"

Важно:
- Каждая модель = подкласс django.db.models.Model
- Каждое поле = экземпляр класса Field
- __str__() метод важен для отображения объектов

### Активация приложения
# mysite/settings.py
```
INSTALLED_APPS = [
    "polls.apps.PollsConfig",  # добавить в начало
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```
### Создание и применение миграций
# Создать миграции
```
python manage.py makemigrations polls
```
# Просмотреть SQL миграции (опционально)
```
python manage.py sqlmigrate polls 0001
```
# Применить миграции
```
python manage.py migrate
```
Трёхшаговый процесс изменения моделей:
1. Изменить модели в models.py
2. python manage.py makemigrations - создать миграции
3. python manage.py migrate - применить к БД

### Работа с API в shell
python manage.py shell
```
# Импорты
from polls.models import Question, Choice
from django.utils import timezone

# Создание объекта
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()

# Доступ к полям
q.id  # 1
q.question_text  # "What's new?"
q.pub_date

# Изменение
q.question_text = "What's up?"
q.save()

# Получение всех объектов
Question.objects.all()

# Фильтрация
Question.objects.filter(id=1)
Question.objects.filter(question_text__startswith="What")

# Получение по первичному ключу
Question.objects.get(pk=1)

# Работа со связями
q = Question.objects.get(pk=1)

# Получить связанные объекты
q.choice_set.all()

# Создать связанный объект
q.choice_set.create(choice_text="Not much", votes=0)
q.choice_set.create(choice_text="The sky", votes=0)
c = q.choice_set.create(choice_text="Just hacking again", votes=0)

# Обратная связь
c.question

# Количество
q.choice_set.count()

# Удаление
c = q.choice_set.filter(choice_text__startswith="Just hacking")
c.delete()
```
### Django Admin

#### Создание суперпользователя
```
python manage.py createsuperuser
```
```
Username: admin
Email address: admin@example.com
Password: **********
Password (again): *********
```
#### Запуск сервера и вход
```
python manage.py runserver
```
http://127.0.0.1:8000/admin/

#### Регистрация модели в админке
# polls/admin.py
```
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```
Теперь Question отображается в админке и можно:
- Просматривать список объектов
- Редактировать объекты
- Удалять объекты
- Создавать новые объекты

---

## Часть 3: Представления и шаблоны

### Добавление представлений
# polls/views.py
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")

def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
### URLconf с параметрами
# polls/urls.py
```
from django.urls import path
from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
Как работает:
- <int:question_id> - захватывает часть URL как integer
- question_id - имя параметра, передаваемого в функцию view
- int - конвертер типа

### Представления с использованием БД
# polls/views.py
```
from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```
Проблема: дизайн жёстко закодирован в Python коде

### Использование шаблонов

Создать структуру:
polls/templates/polls/index.html

Почему двойная вложенность polls/templates/polls/?
- Django ищет шаблоны в каталоге templates каждого приложения
- Двойная вложенность создаёт namespace, чтобы избежать конфликтов имён

# polls/templates/polls/index.html
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}

### Обновление представления для использования шаблона
# polls/views.py
from django.http import HttpResponse
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {"latest_question_list": latest_question_list}
    return HttpResponse(template.render(context, request))
```
### Shortcut: render()
# polls/views.py
```
from django.shortcuts import render
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```
render() принимает:
1. request object
2. имя шаблона
3. словарь контекста (опционально)

### Вызов ошибки 404
# polls/views.py
```
from django.http import Http404
from django.shortcuts import render
from .models import Question

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})

### Shortcut: get_object_or_404()
# polls/views.py
from django.shortcuts import get_object_or_404, render
from .models import Question

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```
get_object_or_404() принимает:
1. модель Django
2. произвольное количество keyword аргументов для get()

Также есть: get_list_or_404() - использует filter() вместо get()

### Шаблон detail
# polls/templates/polls/detail.html
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```
Система шаблонов:
- {{ question.question_text }} - доступ к атрибутам
- question.choice_set.all - вызов метода (без скобок)
- {% for %} {% endfor %} - циклы
- {% if %} {% endif %} - условия

### Удаление жёстко закодированных URL
Было:
```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
Стало:
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
Преимущества:
- Если изменить URL в urls.py, не нужно менять шаблоны
- {% url %} ищет определение URL по имени (name="detail")

### URL namespaces
Проблема: если несколько приложений имеют view с именем "detail", как различить?

Решение - добавить namespace:

# polls/urls.py
```
from django.urls import path
from . import views

app_name = "polls"  # добавить namespace
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
Обновить шаблон:
```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```
Теперь используется: polls:detail вместо просто detail

---

## Ключевые команды Django

### Управление проектом
```
django-admin startproject mysite djangotutorial  # создать проект
python manage.py startapp polls                  # создать приложение
python manage.py runserver                       # запустить сервер
python manage.py runserver 8080                  # на другом порту
```
### База данных и миграции
```
python manage.py migrate                         # применить миграции
python manage.py makemigrations polls            # создать миграции
python manage.py sqlmigrate polls 0001           # посмотреть SQL
python manage.py check                           # проверить проект
```
### Администрирование
```
python manage.py createsuperuser                 # создать суперпользователя
python manage.py shell                           # Django shell
```
### Полезные паттерны

# Получение объекта или 404
```
question = get_object_or_404(Question, pk=question_id)
```
# Получение списка или 404
```
choices = get_list_or_404(Choice, question=question)
```
# Рендер шаблона
```
return render(request, "template.html", context)
```
# Фильтрация
```
Question.objects.filter(pub_date__year=2025)
Question.objects.filter(question_text__startswith="What")
```
# Сортировка
```
Question.objects.order_by("-pub_date")[:5]  # последние 5, по убыванию
```
# Связи
```
question.choice_set.all()                    # все выборы вопроса
question.choice_set.create(...)              # создать связанный объект
question.choice_set.count()                  # количество
choice.question                              # обратная связь
```
---

## Основные концепции Django

### MTV паттерн
- Model (модель) - структура данных, БД
- Template (шаблон) - представление, HTML
- View (представление) - логика, обработка запросов

### Принцип DRY (Don't Repeat Yourself)
- Определяйте структуру данных в одном месте (models.py)
- Django автоматически генерирует API, админку, формы

### URL диспетчеризация
1. Запрос приходит на URL
2. Django смотрит ROOT_URLCONF
3. Ищет совпадение в urlpatterns
4. При include() передаёт остаток URL дальше
5. Вызывает соответствующий view с параметрами

### Философия разработки
- Слабая связанность компонентов
- Модульность (приложения независимы)
- Явное лучше неявного
- Автоматизация рутинных задач

---

## Часть 4: Формы и Generic Views

### Создание формы

<!-- polls/templates/polls/detail.html -->
```
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```
Важные моменты:
- method="post" - использовать для изменения данных на сервере
- {% csrf_token %} - защита от межсайтовой подделки запросов (CSRF)
- forloop.counter - счетчик итераций цикла
- action="{% url 'polls:vote' question.id %}" - URL для обработки формы

### Обработка POST-запроса

# polls/views.py
```
from django.db.models import F
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from .models import Choice, Question

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        # Повторно отобразить форму с ошибкой
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes = F("votes") + 1
        selected_choice.save()
        # Всегда возвращать HttpResponseRedirect после POST
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```
Ключевые концепции:
- request.POST - словарь с POST-данными (значения всегда строки)
- request.POST["choice"] - вызовет KeyError если нет данных
- F("votes") + 1 - атомарное увеличение в БД
- HttpResponseRedirect - всегда после успешного POST
- reverse() - генерация URL по имени view

### Представление результатов

# polls/views.py
```
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})

# polls/templates/polls/results.html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
Фильтр pluralize:
- Добавляет "s" если votes != 1
- 1 vote, 2 votes

### Generic Views (Общие представления)

Проблема: много повторяющегося кода для типичных операций
Решение: использовать встроенные generic views

#### Изменение URLconf

# polls/urls.py
```
from django.urls import path
from . import views

app_name = "polls"
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
Важно: параметр изменился с <question_id> на <pk> (primary key)

#### Новые представления на основе классов

# polls/views.py
```
from django.views import generic
from .models import Question

class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Возвращает последние 5 опубликованных вопросов."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"


class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"
```
Типы Generic Views:
- ListView - отображение списка объектов
- DetailView - отображение деталей одного объекта

Атрибуты:
- model - модель для работы
- template_name - имя шаблона (иначе <app>/<model>_detail.html)
- context_object_name - имя переменной в контексте
- get_queryset() - метод для кастомизации запроса

Автоматические имена:
- DetailView по умолчанию: <app>/<model>_detail.html
- ListView по умолчанию: <app>/<model>_list.html
- Переменная контекста DetailView: question (имя модели)
- Переменная контекста ListView: question_list (автоматически)

---

## Часть 5: Тестирование

### Зачем нужны тесты?

Преимущества:
- Экономят время (автоматизация проверок)
- Предотвращают проблемы (выявляют регрессии)
- Делают код привлекательнее (другие разработчики доверяют коду с тестами)
- Помогают командам работать вместе

### Обнаружение бага

Проблема: was_published_recently() возвращает True для будущих вопросов

python manage.py shell
```
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> future_question.was_published_recently()
True  # Неправильно! Будущее != недавно
```

### Создание теста

# polls/tests.py
```
import datetime
from django.test import TestCase
from django.utils import timezone
from .models import Question

class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() возвращает False для вопросов 
        с pub_date в будущем.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)

Структура теста:
- Наследуемся от django.test.TestCase
- Методы тестов начинаются с test_
- Используем методы проверки: assertIs(), assertEqual(), и т.д.
```
### Запуск тестов

python manage.py test polls
```
Результат:
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_was_published_recently_with_future_question
----------------------------------------------------------------------
Traceback (most recent call last):
  ...
AssertionError: True is not False
----------------------------------------------------------------------
Ran 1 test in 0.001s
FAILED (failures=1)
Destroying test database for alias 'default'...
```
Что происходит:
1. manage.py test polls ищет тесты в приложении polls
2. Находит подкласс TestCase
3. Создает специальную тестовую БД
4. Ищет методы, начинающиеся с test
5. Запускает тесты
6. Удаляет тестовую БД

### Исправление бага

# polls/models.py
```
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```
Теперь тест проходит:
```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s
OK
Destroying test database for alias 'default'...
```
### Более полные тесты

# polls/tests.py
```
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() возвращает False для вопросов
    старше 1 дня.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() возвращает True для вопросов
    за последний день.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```
### Тестирование представлений

#### Django Test Client

python manage.py shell
```
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()  # настройка тестового окружения

>>> from django.test import Client
>>> client = Client()

>>> response = client.get("/")
>>> response.status_code
404

>>> from django.urls import reverse
>>> response = client.get(reverse("polls:index"))
>>> response.status_code
200
>>> response.content
>>> response.context["latest_question_list"]
```
#### Улучшение представления

Проблема: в списке показываются вопросы с pub_date в будущем

# polls/views.py
```
from django.utils import timezone

class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """
        Возвращает последние 5 опубликованных вопросов 
        (не включая те, что запланированы на будущее).
        """
        return Question.objects.filter(
            pub_date__lte=timezone.now()
        ).order_by("-pub_date")[:5]

pub_date__lte=timezone.now() - меньше или равно текущему времени (less than or equal)
```
#### Тесты для представлений

# polls/tests.py
```
from django.urls import reverse

def create_question(question_text, days):
    """
    Создать вопрос с заданным текстом и датой публикации
    со смещением в days дней от текущего момента
    (отрицательное для прошлого, положительное для будущего).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)


class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        Если вопросов нет, отображается соответствующее сообщение.
        """
        response = self.client.get(reverse("polls:index"))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_past_question(self):
        """
        Вопросы с pub_date в прошлом отображаются на странице индекса.
        """
        question = create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_future_question(self):
        """
        Вопросы с pub_date в будущем не отображаются на странице индекса.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_future_question_and_past_question(self):
        """
        Даже если есть и прошлые, и будущие вопросы, 
        отображаются только прошлые.
        """
        question = create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_two_past_questions(self):
        """
        Страница индекса может отображать несколько вопросов.
        """
        question1 = create_question(question_text="Past question 1.", days=-30)
        question2 = create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question2, question1],
        )
```
Методы тестирования:
- self.client.get() - выполнить GET-запрос
- self.assertEqual() - проверить равенство
- self.assertContains() - проверить наличие текста в ответе
- self.assertQuerySetEqual() - сравнить QuerySet

#### Тестирование DetailView

# polls/views.py
```
class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"
    
    def get_queryset(self):
        """
        Исключает вопросы, которые еще не опубликованы.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```
# polls/tests.py
```
class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        DetailView вопроса с pub_date в будущем возвращает 404.
        """
        future_question = create_question(question_text="Future question.", days=5)
        url = reverse("polls:detail", args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        DetailView вопроса с pub_date в прошлом отображает текст вопроса.
        """
        past_question = create_question(question_text="Past Question.", days=-5)
        url = reverse("polls:detail", args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
```
### Правила тестирования

При тестировании чем больше, тем лучше:
- Тесты можно написать один раз и забыть
- Тесты продолжают работать при развитии программы
- Избыточность в тестах - это хорошо

Хорошие практики:
- Отдельный TestClass для каждой модели или представления
- Отдельный метод для каждого набора условий
- Названия методов, описывающие их функцию

---

## Часть 6: Статические файлы

### Настройка статических файлов

Создать структуру:
polls/static/polls/style.css

Почему двойная вложенность polls/static/polls/?
- Django ищет static в каталоге каждого приложения
- Двойная вложенность создает namespace

### Создание CSS

# polls/static/polls/style.css
```
li a {
    color: green;
}
```
### Подключение в шаблоне

# polls/templates/polls/index.html
```
{% load static %}

<link rel="stylesheet" href="{% static 'polls/style.css' %}">
```
Тег {% static %}:
- Генерирует абсолютный URL к статическому файлу
- Работает только в шаблонах Django

### Добавление изображения

Создать: polls/static/polls/images/background.png

# polls/static/polls/style.css
```
body {
    background: white url("images/background.png") no-repeat;
}
```
ВАЖНО:
- Тег {% static %} НЕ работает в CSS
- В статических файлах использовать относительные пути
- url("images/background.png") - относительно CSS файла

### Настройки статических файлов

# mysite/settings.py
```
STATIC_URL = '/static/'  # URL префикс для статических файлов

STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',  # ищет в app/static/
]
```
Запуск:
```
python manage.py runserver
```
http://localhost:8000/polls/ - зеленые ссылки и фон

---

## Часть 7: Кастомизация админки

### Настройка формы администратора

#### Изменение порядка полей

# polls/admin.py
```
from django.contrib import admin
from .models import Question

class QuestionAdmin(admin.ModelAdmin):
    fields = ["pub_date", "question_text"]

admin.site.register(Question, QuestionAdmin)
```
Теперь pub_date отображается перед question_text

#### Группировка полей (fieldsets)

# polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {"fields": ["question_text"]}),
        ("Date information", {"fields": ["pub_date"]}),
    ]

admin.site.register(Question, QuestionAdmin)
```
Структура fieldsets:
- Кортеж из (заголовок, {"fields": [список_полей]})
- None - без заголовка
- "Date information" - заголовок раздела

### Добавление связанных объектов

#### Inline редактирование

# polls/admin.py
```
from .models import Choice, Question

class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 3  # количество пустых форм

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {"fields": ["question_text"]}),
        ("Date information", {"fields": ["pub_date"], "classes": ["collapse"]}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```
StackedInline - каждое поле на отдельной строке

#### Табличное отображение

# polls/admin.py
```
class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 3
```
TabularInline - компактный табличный формат

### Настройка списка изменений

#### Отображение дополнительных полей

# polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin):
    list_display = ["question_text", "pub_date", "was_published_recently"]
    # ...
```
list_display - список полей для отображения в виде столбцов

#### Улучшение отображения методов

# polls/models.py
```
from django.contrib import admin

class Question(models.Model):
    # ...
    @admin.display(
        boolean=True,
        ordering="pub_date",
        description="Published recently?",
    )
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
```
Декоратор @admin.display:
- boolean=True - отображать как иконку галочки/крестика
- ordering="pub_date" - разрешить сортировку
- description="..." - заголовок столбца

#### Добавление фильтров

# polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin):
    list_display = ["question_text", "pub_date", "was_published_recently"]
    list_filter = ["pub_date"]
    # ...
```
list_filter - добавляет боковую панель с фильтрами

#### Добавление поиска

# polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin):
    list_display = ["question_text", "pub_date", "was_published_recently"]
    list_filter = ["pub_date"]
    search_fields = ["question_text"]
```
search_fields - добавляет поле поиска (использует LIKE запрос)

### Кастомизация шаблонов админки

#### Настройка шаблонов проекта

# mysite/settings.py
```
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / "templates"],  # добавить эту строку
        "APP_DIRS": True,
        # ...
    },
]
```
Создать структуру:
djangotutorial/templates/admin/base_site.html

#### Переопределение base_site.html

Скопировать из: django/contrib/admin/templates/admin/base_site.html

Найти исходники Django:
```
python -c "import django; print(django.__path__)"
```
# djangotutorial/templates/admin/base_site.html
```
{% block branding %}
<div id="site-name">
    <a href="{% url 'admin:index' %}">Polls Administration</a>
</div>
{% if user.is_anonymous %}
  {% include "admin/color_theme_toggle.html" %}
{% endif %}
{% endblock %}
```
Теперь заголовок админки изменен на "Polls Administration"

### Организация шаблонов

Шаблоны проекта: djangotutorial/templates/
- Для общих шаблонов проекта
- Для переопределения шаблонов админки

Шаблоны приложения: polls/templates/
- Для шаблонов конкретного приложения
- Можно переносить между проектами

Приоритет поиска:
1. DIRS (templates/ в проекте)
2. APP_DIRS (templates/ в каждом приложении)

---

## Итоговая структура проекта
```
djangotutorial/
    manage.py
    db.sqlite3
    templates/
        admin/
            base_site.html
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        apps.py
        models.py
        tests.py
        views.py
        urls.py
        migrations/
            __init__.py
            0001_initial.py
        static/
            polls/
                style.css
                images/
                    background.png
        templates/
            polls/
                index.html
                detail.html
                results.html
```
## Ключевые команды

# Тестирование
```
python manage.py test                    # запустить все тесты
python manage.py test polls              # тесты конкретного приложения
python manage.py test polls.tests.QuestionModelTests  # конкретный класс
python manage.py test polls.tests.QuestionModelTests.test_method  # конкретный тест
```
# Статические файлы
```
python manage.py collectstatic           # собрать статику для production
python manage.py findstatic style.css    # найти где находится файл
```
# Админка
```
python manage.py createsuperuser         # создать суперпользователя
```
## Основные концепции

### Формы
- Всегда использовать method="post" для изменения данных
- Всегда добавлять {% csrf_token %}
- После успешного POST - HttpResponseRedirect
- Использовать reverse() вместо хардкода URL

### Generic Views
- ListView - для списков объектов
- DetailView - для деталей одного объекта
- Экономят код для типичных операций
- Легко кастомизируются через атрибуты и методы

### Тестирование
- Тесты экономят время в долгосрочной перспективе
- Пишите тесты для каждой модели и представления
- Используйте TestCase для тестов
- Test Client для тестирования представлений
- Больше тестов = лучше

### Статические файлы
- Хранить в app/static/app/
- Использовать {% static %} в шаблонах
- Относительные пути в CSS/JS
- Namespace для избежания конфликтов

### Админка
- Легко кастомизируется через ModelAdmin
- list_display, list_filter, search_fields
- Inline для связанных объектов
- fieldsets для группировки полей
- Можно переопределять шаблоны
