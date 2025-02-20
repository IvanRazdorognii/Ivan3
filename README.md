# Лабораторная работа №3. Основы работы с базами данных в Laravel
# Цель работы
Познакомиться с основными принципами работы с базами данных в Laravel. Научиться создавать миграции, модели и сиды на основе веб-приложения To-Do App.
№1. Подготовка к работе

![image](https://github.com/user-attachments/assets/8f22c2ad-b577-47ba-a2e8-f53409c7eedb)

## 1. Создание модели Category с миграцией
Запустим команду для создания модели Category и миграции:
```
php artisan make:model Category -m
```
Далее откроем миграцию и добавим нужные поля в файл миграции.Будет находиться в database/migrations:

```
// database/migrations/xxxx_xx_xx_xxxxxx_create_categories_table.php
public function up()
{
    Schema::create('categories', function (Blueprint $table) {
        $table->id();  // первичный ключ
        $table->string('name');  // название категории
        $table->text('description');  // описание категории
        $table->timestamps();  // created_at и updated_at
    });
}
```
## 2. Создание модели Task с миграцией
```
php artisan make:model Task -m
```
Это создаст файл модели Task и миграцию для таблицы tasks. Далее откроем создавшуюся миграцию и добавим нужные поля:
```
public function up()
{
    Schema::create('tasks', function (Blueprint $table) {
        $table->id();  // первичный ключ
        $table->string('title');  // название задачи
        $table->text('description');  // описание задачи
        $table->timestamps();  // created_at и updated_at
    });
}
```
## 3. Создание модели Tag с миграцией
```
php artisan make:model Tag -m
```
Далее откроем миграцию и добавим нужные поля
```
public function up()
{
    Schema::create('tags', function (Blueprint $table) {
        $table->id();  // первичный ключ
        $table->string('name');  // название тега
        $table->timestamps();  // created_at и updated_at
    });
}
```
## 4. Модели с $fillable для массового заполнения данных
Добавьте свойство $fillable в каждую модель для массового заполнения. 

// app/Models/Category.php
```
class Category extends Model
{
    protected $fillable = ['name', 'description'];
}
```
// app/Models/Task.php
```
class Task extends Model
{
    protected $fillable = ['title', 'description'];
}
```

// app/Models/Tag.php
```
class Tag extends Model
{
    protected $fillable = ['name'];
}
```
## 5. Запуск миграции

![image](https://github.com/user-attachments/assets/1caa28c3-9931-4034-89f6-825d17179645)
