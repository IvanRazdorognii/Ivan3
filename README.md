
# Лабораторная работа №3. Основы работы с базами данных в Laravel
# Цель работы
Познакомиться с основными принципами работы с базами данных в Laravel. Научиться создавать миграции, модели и сиды на основе веб-приложения To-Do App.
№1. Подготовка к работе

![image](https://github.com/user-attachments/assets/8f22c2ad-b577-47ba-a2e8-f53409c7eedb)
## №2. Создание моделей и миграций

### 1. Создание модели Category с миграцией
Запустим команду для создания модели Category и миграции:
```
php artisan make:model Category -m
```
Далее откроем миграцию и добавим нужные поля в файл миграции.Будет находиться в database/migrations:

``` php
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
### 2. Создание модели Task с миграцией
```
php artisan make:model Task -m
```
Это создаст файл модели Task и миграцию для таблицы tasks. Далее откроем создавшуюся миграцию и добавим нужные поля:
```php
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
### 3. Создание модели Tag с миграцией
```
php artisan make:model Tag -m
```
Далее откроем миграцию и добавим нужные поля
```php
public function up()
{
    Schema::create('tags', function (Blueprint $table) {
        $table->id();  // первичный ключ
        $table->string('name');  // название тега
        $table->timestamps();  // created_at и updated_at
    });
}
```
### 4. Модели с $fillable для массового заполнения данных
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
### 5. Запуск миграции

![image](https://github.com/user-attachments/assets/1caa28c3-9931-4034-89f6-825d17179645)

## №3. Связь между таблицами
 ### Создайте миграцию для добавления поля category_id в таблицу task.
```
php artisan make:migration add_category_id_to_tasks_table --table=tasks
```
### Далее откроем файл миграции, и добавим в структуру таблицы поле category_id и внешний ключ:
```php
public function up()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->unsignedBigInteger('category_id')->nullable();  // добавление поля category_id
        $table->foreign('category_id')->references('id')->on('categories')->onDelete('set null');  // внешний ключ
    });
}

public function down()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropForeign(['category_id']);  // удаление внешнего ключа
        $table->dropColumn('category_id');  // удаление поля category_id
    });
}
```
### Создайте промежуточную таблицу для связи многие ко многим между задачами и тегами:
```
php artisan make:migration create_task_tag_table
```
### Далее откроем файл миграции,и добавим структуру для связи задачи и тега:
```php
public function up()
{
    Schema::create('task_tag', function (Blueprint $table) {
        $table->id();
        $table->unsignedBigInteger('task_id');  // внешний ключ для задачи
        $table->unsignedBigInteger('tag_id');   // внешний ключ для тега
        $table->timestamps();

        $table->foreign('task_id')->references('id')->on('tasks')->onDelete('cascade');
        $table->foreign('tag_id')->references('id')->on('tags')->onDelete('cascade');
    });
}

public function down()
{
    Schema::dropIfExists('task_tag');
}
```
### 3. Запуск миграций

![image](https://github.com/user-attachments/assets/720c381b-9a0b-43c1-93c8-916e0de92f51)

## №4. Связи между моделями
### Добавьте отношения в модель Category (Категория может иметь много задач)
```php
// app/Models/Category.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    // Определение отношения "один ко многим" с таблицей tasks
    public function tasks()
    {
        return $this->hasMany(Task::class);
    }

    // Разрешение массового заполнения
    protected $fillable = ['name', 'description'];
}
```
### Добавьте отношения в модель Task
```php
// app/Models/Task.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    // Определение отношения "многие к одному" с категорией
    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    // Определение отношения "многие ко многим" с тегами
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }

    // Разрешение массового заполнения
    protected $fillable = ['title', 'description', 'category_id'];
}
```

### Добавьте отношения в модель Tag (Тег может быть прикреплен к многим задачам)
```php
// app/Models/Tag.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Tag extends Model
{
    // Определение отношения "многие ко многим" с задачами
    public function tasks()
    {
        return $this->belongsToMany(Task::class);
    }

    // Разрешение массового заполнения
    protected $fillable = ['name'];
}

```
## №5. Создание фабрик и сидов
```
php artisan make:factory CategoryFactory --model=Category
```
### Далее определим структуру для генерации категорий
```php
// database/factories/CategoryFactory.php

namespace Database\Factories;

use App\Models\Category;
use Illuminate\Database\Eloquent\Factories\Factory;

class CategoryFactory extends Factory
{
    protected $model = Category::class;

    public function definition()
    {
        return [
            'name' => $this->faker->word(),
            'description' => $this->faker->sentence(),
        ];
    }
}

```
### 1.2 Фабрика для модели Task
Запустите команду для создания фабрики:

```
php artisan make:factory TaskFactory --model=Task
```

Откроем файл фабрики database/factories/TaskFactory.php и определим структуру данных:
```php
namespace Database\Factories;

use App\Models\Task;
use App\Models\Category;
use App\Models\Tag;
use Illuminate\Database\Eloquent\Factories\Factory;

class TaskFactory extends Factory
{
    protected $model = Task::class;

    public function definition()
    {
        return [
            'title' => $this->faker->sentence(),
            'description' => $this->faker->paragraph(),
            'category_id' => Category::factory(), // Автоматическое создание категории
        ];
    }
}
```
### 1.3 Фабрика для модели Tag
Запустиv команду для создания фабрики:
```
php artisan make:factory TagFactory --model=Tag
```
Откроем файл фабрики database/factories/TagFactory.php и определим структуру данных:
```php
namespace Database\Factories;

use App\Models\Tag;
use Illuminate\Database\Eloquent\Factories\Factory;

class TagFactory extends Factory
{
    protected $model = Tag::class;

    public function definition()
    {
        return [
            'name' => $this->faker->word(),
        ];
    }
}
```
## 2. Создайте сиды (seeders) для заполнения таблиц начальными данными для моделей: Category, Task, Tag.

### 2.1 Сид для модели Category
Запустим команду для создания сидера:
```
php artisan make:seeder CategorySeeder
```
Откроем файл database/seeders/CategorySeeder.php и добавим код для заполнения таблицы категориями:
```php
namespace Database\Seeders;

use App\Models\Category;
use Illuminate\Database\Seeder;

class CategorySeeder extends Seeder
{
    public function run()
    {
        // Создаем 10 категорий
        Category::factory(10)->create();
    }
}
```
### 2.2 Сид для модели Task
Запустим команду для создания сидера:
```
php artisan make:seeder TaskSeeder
```
Откроем файл database/seeders/TaskSeeder.php и добавим код для заполнения таблицы задачами:
```php
namespace Database\Seeders;

use App\Models\Task;
use Illuminate\Database\Seeder;

class TaskSeeder extends Seeder
{
    public function run()
    {
        // Создаем 50 задач
        Task::factory(50)->create();
    }
}
```
### 2.3 Сид для модели Tag
Запустим команду для создания сидера:
```
php artisan make:seeder TagSeeder
```
Откроем файл database/seeders/TagSeeder.php и добавим код для заполнения таблицы тегами:
```php
namespace Database\Seeders;

use App\Models\Tag;
use Illuminate\Database\Seeder;

class TagSeeder extends Seeder
{
    public function run()
    {
        // Создаем 20 тегов
        Tag::factory(20)->create();
    }
}
```
### 3. Обновление DatabaseSeeder

Откроем файл database/seeders/DatabaseSeeder.php и добавим вызов всех созданных сидеров:
```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        // Запуск всех сидеров
        $this->call([
            CategorySeeder::class,
            TaskSeeder::class,
            TagSeeder::class,
        ]);
    }
}
```
### 4. Запуск сидеров
После того как все сидеры созданы, выполним команду для заполнения таблиц начальными данными:
```
php artisan db:seed
```
![image](https://github.com/user-attachments/assets/3e5f861f-99e7-4cc9-84c7-1403b9c031b9)

## №6. Работа с контроллерами и представлениями
### Откройте контроллер TaskController (app/Http/Controllers/TaskController.php).
Обновите метод index для получения списка задач из базы данных.
```php
// app/Http/Controllers/TaskController.php

namespace App\Http\Controllers;

use App\Models\Task;

class TaskController extends Controller
{
    public function index()
    {
        // Получаем все задачи с категориями и тегами
        $tasks = Task::with(['category', 'tags'])->get();
        
        return view('tasks.index', compact('tasks'));
    }
}

```
### 2. Обновление метода show
```php
public function show($id)
{
    // Получаем задачу с категорией и тегами по её ID
    $task = Task::with(['category', 'tags'])->findOrFail($id);

    return view('tasks.show', compact('task'));
}
```
### 3. Обновление представлений
tasks/index.blade.php — Список задач
В этом представлении отобразим список всех задач и их категории и теги.
```
@extends('layouts.app')

@section('content')
    <h1>Список задач</h1>

    <ul>
        @foreach($tasks as $task)
            <li>
                <h3>{{ $task->title }}</h3>
                <p>Категория: {{ $task->category->name }}</p>
                <p>Теги: 
                    @foreach($task->tags as $tag)
                        <span>{{ $tag->name }}</span>
                    @endforeach
                </p>
                <a href="{{ route('tasks.show', $task->id) }}">Просмотреть задачу</a>
            </li>
        @endforeach
    </ul>
@endsection
```
### tasks/show.blade.php — Просмотр отдельной задачи
В этом представлении отобразим детальную информацию о задаче.
``` php
@extends('layouts.app')

@section('content')
    <h1>{{ $task->title }}</h1>
    <p>Описание: {{ $task->description }}</p>
    <p>Категория: {{ $task->category->name }}</p>
    <p>Теги: 
        @foreach($task->tags as $tag)
            <span>{{ $tag->name }}</span>
        @endforeach
    </p>
    <a href="{{ route('tasks.edit', $task->id) }}">Редактировать</a>
@endsection
```
### 4. Обновление метода create для отображения формы создания задачи
Метод create должен отображать форму для создания задачи.
```php
public function create()
{
    return view('tasks.create');
}
```
### Форма для создания задачи (например, в tasks/create.blade.php)
```php
@extends('layouts.app')

@section('content')
    <h1>Создание задачи</h1>

    <form action="{{ route('tasks.store') }}" method="POST">
        @csrf
        <label for="title">Название задачи:</label>
        <input type="text" name="title" id="title" required>
        
        <label for="description">Описание задачи:</label>
        <textarea name="description" id="description" required></textarea>

        <label for="category_id">Категория:</label>
        <select name="category_id" id="category_id">
            <!-- Сюда нужно добавить все категории -->
        </select>

        <label for="tags">Теги:</label>
        <input type="text" name="tags" id="tags">

        <button type="submit">Создать задачу</button>
    </form>
@endsection
```
### 5. Обновление метода store для сохранения задачи
Метод store должен сохранять новую задачу в базе данных.
``` php
public function store(Request $request)
{
    // Валидация данных
    $request->validate([
        'title' => 'required|string|max:255',
        'description' => 'required|string',
        'category_id' => 'required|exists:categories,id',
    ]);

    // Сохранение задачи
    $task = Task::create([
        'title' => $request->input('title'),
        'description' => $request->input('description'),
        'category_id' => $request->input('category_id'),
    ]);

    // Привязка тегов
    $task->tags()->sync($request->input('tags'));

    return redirect()->route('tasks.index');
}
```
### 6. Обновление метода edit для отображения формы редактирования задачи
Метод edit должен показывать форму для редактирования задачи.
``` php
public function edit($id)
{
    $task = Task::findOrFail($id);
    return view('tasks.edit', compact('task'));
}
```
### Форма для редактирования задачи (например, в tasks/edit.blade.php)
```php
@extends('layouts.app')

@section('content')
    <h1>Редактирование задачи</h1>

    <form action="{{ route('tasks.update', $task->id) }}" method="POST">
        @csrf
        @method('PUT')

        <label for="title">Название задачи:</label>
        <input type="text" name="title" id="title" value="{{ $task->title }}" required>
        
        <label for="description">Описание задачи:</label>
        <textarea name="description" id="description" required>{{ $task->description }}</textarea>

        <label for="category_id">Категория:</label>
        <select name="category_id" id="category_id">
            <!-- Сюда нужно добавить все категории -->
        </select>

        <label for="tags">Теги:</label>
        <input type="text" name="tags" id="tags" value="{{ $task->tags->pluck('name')->implode(', ') }}">

        <button type="submit">Сохранить изменения</button>
    </form>
@endsection
```
### 7. Обновление метода update для сохранения изменений
Метод update должен сохранять изменения в базе данных.
``` php
public function update(Request $request, $id)
{
    $task = Task::findOrFail($id);

    $request->validate([
        'title' => 'required|string|max:255',
        'description' => 'required|string',
        'category_id' => 'required|exists:categories,id',
    ]);

    $task->update([
        'title' => $request->input('title'),
        'description' => $request->input('description'),
        'category_id' => $request->input('category_id'),
    ]);

    $task->tags()->sync($request->input('tags'));

    return redirect()->route('tasks.index');
}
```
### 8. Обновление метода destroy для удаления задачи
Метод destroy должен удалять задачу из базы данных.
```php
public function destroy($id)
{
    $task = Task::findOrFail($id);
    $task->delete();

    return redirect()->route('tasks.index');
}
```
![image](https://github.com/user-attachments/assets/59c455f2-76b5-42f3-813b-6db89bbc5ac5)

![image](https://github.com/user-attachments/assets/034a53e9-07be-405f-995f-1dcb4c7c8ed5)

![image](https://github.com/user-attachments/assets/e72bd31b-02d7-44a9-a01f-5425851624da)
