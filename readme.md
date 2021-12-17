# Шпаргалка по django

### Уроки: https://github.com/selfedu-rus/django-lessons
### Видео к урокам: https://www.youtube.com/playlist?list=PLA0M1Bcd0w8xO_39zZll2u1lz_Q-Mwn1F

## Классы, Модели и Базы данных:
 поле для автоматической записи даты и времени создания записи в таблице БД:
```
time_create = models.DateTimeField(auto_now_add=True)
```
### Аргументы поля.
Ссылка на документацию https://docs.djangoproject.com/en/2.2/ref/models/fields/

для обновления записи:
```
auto_now=True
```
для индексации поля (для более быстрого поиска по таблице)
```
db_index = true
```
разрешить оставлять поле без значения:
```
null=True
```
разрешить оставлять значение пустым, для текстовых и строковых полей:
```
blank=True
```
 ## Созданеи записей в БД
 ### документация по запросам orm Django https://djbook.ru/rel3.0/topics/db/queries.html
 ### "Ленивое" наполнение БД. 
 Создаем экземпляр класса
 ```
 user1 = Users()
 ```
  в разны частях приложения наполняем его:
 ```
 user1.name = 'Misha'
 user1.email = 'misha@mail.ru'
 ```
  в нужный нам момент сохраняем, только в этот момент объект класса модели создается в БД как запись:
 ```
 user1.save()
 ```
 ### Второй вариант. С помощью стандартного метода objects.create() 
 сразу создаем запись в БД и присваиваем ее экземпляру класса:
```
user2 = Users.objects.create(name='BorisB', email='borya@mail.ru')
```
или вообще не создавать экземляр класса, а только запись в БД:
```
Users.objects.create(name='BorisB', email='borya@mail.ru')
```
## Чтение записей из БД
 считать сразу все записи таблицы (если их немного!)
```
Users.objects.all()
```
чтобы .all() отобразил читабельные названия/имена записей, в моделе должен быть метод str:
```
    def __str__(self):
        return self.name
```
 считываем все записи таблицы, помещаем их в список, затем можем обращаться к отдельным полям элементов списка:
```
u = Users.objects.all()
print(u[0].name, u[1].name)
```
получение первый 10 записей:
```
Users.objects.all()[:10]
```
получение последних 10 записей. сначала делаем обратную сортировку, потом берем 10 записей, выполняется за один sql запрос:
```
Users.objects.all().order_by('-id')[:10]
```
### Используем метод filter для выборки по какому-либо полю (можно несколько условий через запятую). результат выполнение - список объектов класса Users
```
Users.objects.filter(email='borya@mail.ru')
```
 фильтр по именованному параметру pk (primary key, в общем случае == id). например, все записи поместить в u1 с id >=5 и все записи <= 10 поместить в u2:
```
u1 = Users.objects.filter(pk__gte=5)
u2 = Users.objects.filter(pk__lte=5)
```
### Используем метод exclude  - выбрать все записи кроме тех, что подпадают под усоловие. например, все записи, кроме id=1
```
Users.objects.exclude(pk=1)
```
если нам нужно получить какую-то определенную запись по какому-либо определенному полю, то лучше использовать метод get. например, получим запись с именем BorisB:
```
Users.objects.get(name='BorisB')
```
### !Если в результате выполнение get будет получено несколько записей или не будет ни одной записи, то будет сгенерировано исключение!
## Изменение записей в таблице
счтитаем запись таблицы через метод get, запишем ее в объект класса модели, изменим необходимые поля и сохраним:
```
user = Users.objects.get(name='BorisB')
user.name = 'Borisych'
user.save()
```
## Удаление записей таблицы
выбрать фильтром записи в список bad_users, затем удалить методом delete:
```
bad_users = Users.objects.filter(pk__gte=2)
bad_users.delete()
```
## Отображение содержимого таблицы БД на странице через шаблон
на примере модели Posts. 

### models.py:
```
class Posts(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField(blank=True)
    photo = models.ImageField(upload_to="photos/%Y/%m/%d/")
    time_create = models.DateTimeField(auto_now_add=True)
    time_update = models.DateTimeField(auto_now=True)
```
создаем отдельный класс отображения таблицы Posts как дочерний класс ListView. 

### views.py:
```
from .models import Posts
from django.views.generic.list import ListView

class PostsView(ListView):
    model = Posts
    template_name = 'blog/index.html'
    context_object_name = 'posts'

def index(request):
    posts = Posts.objects.all()
    return render(request, 'blog/index.html')
```
для вывода конкретного поста с id = some_post_id ожно использовать стандартную функцию джанго, которая выводит ошибку 404, если запись в базе не будет найдена
```
from django.shortcuts import get_object_or_404
post = get_object_or_404(Posts, pk=some_post_id)
```
### index.html
```
<!DOCTYPE html>
<body>
    <h1>Вывод содержимого таблицы Posts</h1>
    {% for p in posts %}
        <h2>{{ p.title }}</h2>
        <p>{{ p.content}}</p>
        <p>{{p.time_update}}</p>
    {% endfor %}
</body>
</html>
```
## Файл настроек settyngs.py

папка temlates в корне проекта:
```
TEMPLATE_DIR = os.path.join(BASE_DIR, 'templates')
```
префикс URL-адреса для статических файлов:
```
STATIC_URL = '/static/'
```
путь к общей статической папке:
```
STATIC_ROOT = os.path.join(BASE_DIR, "static")
```
список дополнительных (нестандартных) путей, обычно используется для отладки:
```
STATICFILES_DIRS = []
```
спискок адресов и доменов, с которых разрешен вход:
```
ALLOWED_HOSTS = ['127.0.0.1']
```
путь к медиа контенту, папка media в корне проекта:
```
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```
префикс URL-адреса для медиа файлов:
```
MEDIA_URL = '/media/
```
## Отображение формы, запись в БД из формы

### Шаблон для отображения формы
Передать POST запрос функции представления addpage (простейший вариант):
```
<form action="{% url 'addpage' %}" method="post">
    {% csrf_token %}
    {{ form.as_p }}
<button type="submit">Добавить</button>
</form>

```
Вариант через цикл (стилизованныцй), с полем для вывода ошибок:
```
<form action="{% url 'addpage' %}" method="post">
    {% csrf_token %}
    {% for f in form %}
        <p><label class="form-label" for="{{ f.id_for_label }}">{{ f.label }}: </label>{{ f }}</p>
        <div class="form-error">{{ f.errors }}</div>
    {% endfor %}
<button type="submit">Добавить</button>
</form>
```
### Описание класса формы
Прописываем форму для добавление записи поста класса Post в forms.py:
```
from django import forms
from .models import *

class AddPostForm(forms.Form):
    title = forms.CharField(max_length=200, label='Заголовок')
    slug = forms.SlugField(max_length=100, label='URL')
    content = forms.CharField(widget=forms.Textarea(attrs={'cols': 60, 'rows': 10}), label='Текст')
    is_public = forms.BooleanField(label='Опубликовать', required=False, initial=True)
    cat = forms.ModelChoiceField(queryset=Category.objects.all(), label='Категория', empty_label='Не выбрано')
```
Также в описании класса формы можно сразу определить стиль каждого поля с помощью параметра widget:
```
title = forms.CharField(max_length=200, label='Заголовок', widget=forms.TextInput(attrs={'class':'form-input'}))
```
Но правильнее связать форму напрямую с моделью:
```
class AddPostForm(forms.ModelForm):
    class Meta:
        model = Posts
        fields = '__all__'
```
Параметр '__all__' позволяет отобразить все поля, кроме тех, что заполняются автоматически.


### Функция представления формы
views.py, отобразить форму, если запрос не post, в противном случае, проверить на правильность заполнения:
```
def addpage(request):
    if request.method == 'POST':
        form = AddPostForm(request.POST)
        if form.is_valid():
            try:
                Posts.objects.create(**form.cleaned_data)
                return redirect('index')
            except :
                form.add_error(None, 'Ошибка добавления поста')
    else:
        form = AddPostForm()
    return render(request, 'blog/addpage.html', {'form': form, 'menu': menu, 'title': 'Добавление статьи'})
```
При ошибках в заполении Django автоматически выведет подсказки пользователю.Т
Но в случае, когда класс формы наследуется от forms.ModelForm метод Posts.objects.create(**form.cleaned_data) можно заменить на метод save. В этом случае, для корректной загрузки фото, при создании экземпляра формы нужно передать параметр request.FILES. Итого, функция представления формы добавления статьи:
```
def addpage(request):
    if request.method == 'POST':
        form = AddPostForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('index')
    else:
        form = AddPostForm()
    return render(request, 'blog/addpage.html', {'form': form, 'menu': menu, 'title': 'Добавление статьи'})
```
В этом случае, в шаблоне страницы в строке формирования формы добавим enctype="multipart/form-data:
```
<form action="{% url 'add_page' %}" method="post" enctype="multipart/form-data">
```
### Валидаторы данных в полях формы
При вызове метода save() формы сначала выполняются стандартные валидаторы Джанго в соответствии с ограничениями полей модели в models.py. Затем выполняются пользовательские валидаторы.
### Пользовательские валидаторы
Пользовательские валидаторы прописываются в forms.py как методы класса формы:
```
from django.core.exceptions import ValidationError
...
class...

    def clean_title(self):
        title = self.cleaned_data['title']  # присваеваем значение поля title
        if len(title) > 200:  # проверяем длину строки значения
            raise ValidationError('Длина превышает 200 символов')  # сообщение в случе ошибки

        return title
```
Где "clean_" - обязательная часть, дальше title - название поля. 

## Классы представлений
Список встроенных представлений-классов https://djbook.ru/rel1.9/ref/class-based-views/index.html
Наиболее часто используют ListView и ListDetail.

### Класс отображения списка
Класс просмотра списка постов. Выбираются все значения из БД, прописываются методы для передачи динамического контекста и фильтра запроса к БД.
Объявим два класса - список постов (опубликованных) и список постов выбранной категории:
```
from django.views.generic.list import ListView


class PostsView(ListView):
    model = Posts
    template_name = 'blog/posts.html'
    context_object_name = 'posts'

    def get_context_data(self, *, object_list=None, **kwargs):
        context = super().get_context_data(**kwargs)  # получаем весь именованный контекст, который уже сформирован
        context['menu'] = menu  # добавляем свои динамические значения в контекст
        context['title'] = 'Главная страница'
        context['cat_selected'] = 0
        return context  # возвращаем сформированный контекст

    def get_queryset(self):
        return Posts.objects.filter(is_public=True)


class PostsCategory(ListView):
    model = Posts
    template_name = 'blog/posts.html'
    context_object_name = 'posts'
    allow_empty = False

    def get_queryset(self):
        # фильтр по опукликованным статьям и слагу категории
        return Posts.objects.filter(cat__slug=self.kwargs['cat_slug'], is_public=True)  

    def get_context_data(self, *, object_list=None, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'Категория - ' + str(context['posts'][0].cat) 
        context['menu'] = menu
        context['cat_selected'] = context['posts'][0].cat_id
        return context
```
В urls.py добавим строки вызова метода as_view наших классов:
```
path('', views.PostsView.as_view(), name='index'),
path('category/<slug:cat_slug>/', views.PostsCategory.as_view(), name='category'),
```
## QuerySet API
Некоторые полезные функции и медоты для обращения к связанным таблицам БД через API Django.
### Обращение к вторичной таблице по внешнему первичному ключу
```
c = Category.object.get(name='Docker')  # присваеваем переменной объект экземляра Category с именем Docker (подразумевая, что имя уникальное)
c.posts_set.all()  # получаем через переменную класса Category все объекты класса Posts
```
posts_set - получение вторичной модели Posts, чтобы названия метода posts_set заменить на, например, get_posts, нужно дописать дополнительное свойство в ключевом поле модели Posts:
```
cat = models.ForeignKey('Category', on_delete=models.CASCADE, verbose_name="Категория", related_name='get_posts')
```

### Фильтры полей (lookups)
```
Posts.objects.filter(title__contains='Docker')  # выбрать все поля из Posts, поле title который включает 'Docker'
Posts.objects.filter(title__icontains='Docker')  # то же самое, но без учета регистра. !!!В SQLite только для символов ASCI (латинских), в остальных норм
Posts.objects.filter(pk__in=[1, 5, 7])  # выбрать записи с первичным ключем (id) = 1, 3, 7
Posts.objects.filter(pk__in=[1, 5, 7], is_public=True)  # сразу несколько фильтров можно указывать через запятую
Posts.objects.filter(cat__in=[1, 2])  # фильтр по внешнему ключу "cat" (id категории), выбираем все посты первой и второй категории
```
### Использование класса Q для фильтров
& - логическое И.
| - логическое ИЛИ.
~ - логическое НЕ.
Приоритет в следующем порядке: НЕ, И, ИЛИ
```
from django.db.models import Q

Posts.objects.filter(Q(pk__lt=5) | Q(cat_id=2))  # создаются два экземпляра класса Q, между ними оператор ИЛИ. Все записи с pk<=5 или с категорией 2. 
Posts.objects.filter(~Q(cat_id=2))  # все записи, котегория которых не равна 2
```
Можно применять методы к осортированной выборке:
```
Posts.objects.order_by('time_create').first()  # отсортировали по дате создания (поле time_create) и возвратили первую запись
Posts.objects.order_by('-time_create').first()  # по дате создания в обратном порядке и первую запись с конца
Posts.objects.order_by('time_create').last()  # то же, что и предыдущая, но взяли последнюю запись при прямой сортировке
```
Для полей time_create и time_update, содержащих дату и время, существуют специальные методы:
```
Posts.objects.latest('time_create')  # самая последняя запись по дате создания (добавленная самой первой)

Posts.objects.latest('time_create')  # самая новая запись
```
