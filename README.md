## Laravel Sanctum route protection

```sh 
php artisan make:controller TaskController -r
```
```sh 
php artisan make:model Task -m
```

1. the resource controller is created (with the -r flag), i.e. all methods required for CRUD are created, then 
    only the routes need to be defined in api.php:
```sh 
Route::resource('tasks', TaskController::class);
```
2. the model Task with the migration file (-m)


---

### Requests 

```sh 
php artisan make:request StoreUserRequest
```

..you create a request object that you can inject instead of the "normal" request object, basically for encapsulation of 
validating the request within the rules method

..dont forget to switch the return of authorize() to true!

Same for the login:

```sh 
php artisan make:request LoginUserRequest
```

### Filling the tasks table with a factory
- open the migration, add the column: name, description, priority (with default: medium)
- add a column for the constraint to the users table, because 1 User can have multiple tasks:
```sh 
$table->unsignedBigInteger('user_id);
```

..and above the timestamps add the foreign key constraint:
```sh 
$table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
```
..which means: the foreign key "user_id" references the id column on the users table and whenever a user is deleted, all related tasks should be deleted in a cascade fashion.

```sh 
php artisan migrate
```


After that you set up the Task model for massassignment:
```sh 
protected $fillable = ['user_id', 'name', 'description', 'priority'];
```

..and define the relationship with the User model

```sh 
public function user() {
        return $this->belongsTo(User::class);
}
```
..since one task belongs to one user

In the CLI create a factory for the Tasks
```sh 
php artisan make:factory TaskFactory
```

inside TaskFactory -> definition() return an array like:
```sh 
return [
   'user_id' => User::all()->random()->id, // get all users, pick a random one and from that the id
   'name' => $this->faker->unique()->sentence(),
   'description' => $this->faker->text(),
   'priority' => $this->faker->randomElement(['low', 'medium', 'high'])
];
```

With that done, you can create dummy data inside the database with tinker
```
php artisan tinker
```

First create some users
```
User::factory()->times(20)->create();
```

And for the Tasks:
```sh 
App\Models\Task::factory()->times(200)->create();
exit;
```

With that you'd get a collection when you do a GET Request to /tasks in i.e. Postman.
To get a nicely formatted json you can make a TaskResource
```sh 
php artisan make:resource TasksResource
```

In the index method of the TaskController for example you can instead of:
```sh
return Task::all();
```

..which returns a collection, you can use the newly created TasksResource to give it the collection and 

```sh 
return TasksResource::collection(
   Task::where('user_id', Auth::user()->id)->get()
);
```

..so: return a json object with a data property (which is the convention for json api responses) and where the user_id is equal to the currently authenticated user, i.e. the user with the token (which is in the Header of this request) and with ->get() you say: get them all
