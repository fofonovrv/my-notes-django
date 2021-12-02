# Личная шпаргалка по django

### Уроки: https://github.com/selfedu-rus/django-lessons/blob/main/lesson-4-coolsite.zip

## Классы, Модели и Базы данных:
### поле для автоматической записи даты и времени создания записи в таблице БД:
```
time_create = models.DateTimeField(auto_now_add=True)
```
для обновления записи:
```
time_update = models.DateTimeField(auto_now=True)
```
