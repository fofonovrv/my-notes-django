# Личная шпаргалка по django

### Уроки: https://github.com/selfedu-rus/django-lessons

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
  в разны частях приложения наполняем его
 ```
 user1.name = 'Misha'
 user1.email = 'misha@mail.ru'
 ```
  в нужный нам момент сохраняем, только в этот момент объект класса модели создается в БД как запись
 ```
 user1.save()
 ```
 ### Второй вариант. С помощью стандартного метода objects.create() 
 сразу создаем запись в БД и присваиваем ее экземпляру класса
```
user2 = Users.objects.create(name='BorisB', email='borya@mail.ru')
```
или вообще не создавать экземляр класса, а только запись в БД
```
Users.objects.create(name='BorisB', email='borya@mail.ru')
```
## Чтение записей из БД
 считать сразу все записи таблицы (если их немного!)
```
Users.objects.all()
```
чтобы .all() отобразил читабельные названия/имена записей, в моделе должен быть метод str
```
    def __str__(self):
        return self.name
```
 считываем все записи таблицы, помещаем их в список, затем можем обращаться к отдельным полям элементов списка
```
u = Users.objects.all()
print(u[0].name, u[1].name)
```
получение первый 10 записей
```
Users.objects.all()[:10]
```
получение последних 10 записей. сначала делаем обратную сортировку, потом берем 10 записей, выполняется за один sql запрос
```
Users.objects.all().order_by('-id')[:10]
```
### Используем метод filter для выборки по какому-либо полю (можно несколько условий через запятую). результат выполнение - список объектов класса Users
```
Users.objects.filter(email='borya@mail.ru')
```
 фильтр по именованному параметру pk (primary key, в общем случае == id). например, все записи поместить в u1 с id >=5 и все записи <= 10 поместить в u2
```
u1 = Users.objects.filter(pk__gte=5)
u2 = Users.objects.filter(pk__lte=5)
```
### Используем метод exclude  - выбрать все записи кроме тех, что подпадают под усоловие. например, все записи, кроме id=1
```
Users.objects.exclude(pk=1)
```
если нам нужно получить какую-то определенную запись по какому-либо определенному полю, то лучше использовать метод get. например, получим запись с именем BorisB
```
Users.objects.get(name='BorisB')
```
### !Если в результате выполнение get будет получено несколько записей или не будет ни одной записи, то будет сгенерировано исключение!
## Изменение записей в таблице
счтитаем запись таблицы через метод get, запишем ее в объект класса модели, изменим необходимые поля и сохраним
```
user = Users.objects.get(name='BorisB')
user.name = 'Borisych'
user.save()
```
## Удаление записей таблицы
выбрать фильтром записи в список bad_users, затем удалить методом delete
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

### viws.py:
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
