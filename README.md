# laravel13-crud

### Slide 1 — Laravel 13 CRUD with Blade

**Build a simple posts application**

- Create a post
- Read posts
- Update a post
- Delete a post

**Stack:** Laravel 13 · Blade · Tailwind CSS · daisyUI

---

### Slide 2 — How Blade Works

**Display a page:**

Browser → Route → Controller → Model → Blade → Browser

**Submit a form:**

Form → Route → Validate → Save → Redirect

---

### Slide 3 — Create the Table and Model

```bash
php artisan make:model Post -m
php artisan make:controller PostController
```

```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('category_name');
    $table->string('title');
    $table->longText('body');
});
```

```bash
php artisan migrate
```

---

### Slide 4 — Configure the Model

```php
use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Model;

#[Fillable(['category_name', 'title', 'body'])]
class Post extends Model
{
    public $timestamps = false;
}
```

**Fillable:** allows these fields in `create()` and `update()`.

**Timestamps:** disabled because our table has no timestamp columns.

---

### Slide 5 — Map Routes to Actions

| HTTP | URL | Method |
|---|---|---|
| GET | `/posts` | `index` |
| GET | `/posts/create` | `create` |
| POST | `/posts` | `store` |
| GET | `/posts/{id}` | `show` |
| GET | `/posts/{id}/edit` | `edit` |
| PUT | `/posts/{id}` | `update` |
| DELETE | `/posts/{id}` | `destroy` |

```php
Route::get('/posts', [PostController::class, 'index'])
    ->name('posts.index');
```

Register `/posts/create` before `/posts/{id}`.

---

### Slide 6 — Read and Display Posts

**Controller:**

```php
public function index()
{
    $posts = Post::orderByDesc('id')->paginate(10);

    return view('posts.index', compact('posts'));
}
```

**Blade:**

```blade
@forelse ($posts as $post)
    <p>{{ $post->title }}</p>
@empty
    <p>No posts yet.</p>
@endforelse

{{ $posts->links() }}
```

---

### Slide 7 — Create a Post

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'category_name' => ['required', 'string', 'max:255'],
        'title' => ['required', 'string', 'max:255'],
        'body' => ['required', 'string'],
    ]);

    Post::create($validated);

    return redirect()->route('posts.index')
        ->with('status', 'Post created.');
}
```

**Validate → Save → Redirect**

---

### Slide 8 — Blade Form Essentials

```blade
@csrf

<input
    name="title"
    value="{{ old('title', $post->title) }}"
    class="input"
>

@error('title')
    <p class="text-error">{{ $message }}</p>
@enderror
```

- `name` — identifies the submitted field
- `@csrf` — adds the form security token
- `old()` — restores input after validation fails
- `@error` — displays a validation message

---

### Slide 9 — Share Fields, Keep Separate Actions

**Create page:**

```blade
<form action="{{ route('posts.store') }}" method="POST">
    @include('posts.form')
</form>
```

**Edit page:**

```blade
<form action="{{ route('posts.update', $post->id) }}" method="POST">
    @method('PUT')
    @include('posts.form')
</form>
```

The shared partial contains `@csrf`, fields, errors, and a submit button.

---

### Slide 10 — Find and Update a Post

**Load the edit page:**

```php
public function edit(string $id)
{
    $post = Post::findOrFail($id);

    return view('posts.edit', compact('post'));
}
```

**Inside `update()`:**

```php
$post = Post::findOrFail($id);

// Validate the input using the same rules as store().

$post->update($validated);

return redirect()->route('posts.index');
```

`findOrFail()` returns the record or a **404**.

---

### Slide 11 — Delete a Post

**Blade:**

```blade
<form action="{{ route('posts.destroy', $post->id) }}" method="POST">
    @csrf
    @method('DELETE')

    <button class="btn btn-error">Delete</button>
</form>
```

**Controller:**

```php
public function destroy(string $id)
{
    $post = Post::findOrFail($id);
    $post->delete();

    return redirect()->route('posts.index');
}
```

---

### Slide 12 — Live Demo Checklist

1. Create a post.
2. Display its details.
3. Edit and save changes.
4. Demonstrate validation errors.
5. Delete the post.
6. Visit a missing post ID to show a 404.

**Run in separate terminals:**

```bash
php artisan serve
```

```bash
npm run dev
```
