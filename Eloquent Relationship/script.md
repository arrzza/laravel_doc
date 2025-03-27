# Laravel Eloquent Relationships Guide

## 1. One to One (`hasOne`)
### A user has one profile.

### 🛠 Table Structure
#### `users` Table
```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamps();
});
```

#### `profiles` Table
```php
Schema::create('profiles', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->string('bio')->nullable();
    $table->timestamps();
});
```

### 📝 Model
#### `User.php`
```php
public function profile() {
    return $this->hasOne(Profile::class);
}
```

### 📌 Controller Usage
```php
$user = User::with('profile')->find($id);
$profile = $user->profile;
```

---

## 2. One to Many (`hasMany`)
### A post has many comments.

### 🛠 Table Structure
#### `posts` Table
```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->timestamps();
});
```

#### `comments` Table
```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->foreignId('post_id')->constrained()->onDelete('cascade');
    $table->text('comment');
    $table->timestamps();
});
```

### 📝 Model
#### `Post.php`
```php
public function comments() {
    return $this->hasMany(Comment::class);
}
```

### 📌 Controller Usage
```php
$post = Post::with('comments')->find($id);
$comments = $post->comments;
```

---

## 3. Many to One (`belongsTo`)
### A comment belongs to a post.

### 📝 Model
#### `Comment.php`
```php
public function post() {
    return $this->belongsTo(Post::class);
}
```

### 📌 Controller Usage
```php
$comment = Comment::find($id);
$post = $comment->post;
```

---

## 4. Many to Many (`belongsToMany`)
### A user belongs to many roles, and a role belongs to many users.

### 🛠 Table Structure
#### `roles` Table
```php
Schema::create('roles', function (Blueprint $table) {
    $table->id();
    $table->string('name'); // e.g., 'Admin', 'Editor'
    $table->timestamps();
});
```

#### `role_user` Pivot Table
```php
Schema::create('role_user', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->foreignId('role_id')->constrained()->onDelete('cascade');
    $table->timestamps();
});
```

### 📝 Model
#### `User.php`
```php
public function roles() {
    return $this->belongsToMany(Role::class);
}
```

#### `Role.php`
```php
public function users() {
    return $this->belongsToMany(User::class);
}
```

### 📌 Controller Usage
```php
$user = User::with('roles')->find($id);
$roles = $user->roles;
```

---

## 5. Has One Through (`hasOneThrough`)
### A user has one country through a profile.

### 🛠 Table Structure
#### `countries` Table
```php
Schema::create('countries', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->timestamps();
});
```

#### Modify `profiles` Table
```php
Schema::table('profiles', function (Blueprint $table) {
    $table->foreignId('country_id')->constrained()->onDelete('cascade');
});
```

### 📝 Model
#### `User.php`
```php
public function country() {
    return $this->hasOneThrough(Country::class, Profile::class);
}
```

### 📌 Controller Usage
```php
$user = User::with('country')->find($id);
$country = $user->country;
```

---

## 6. Has Many Through (`hasManyThrough`)
### A country has many posts through users.

### 🛠 Table Structure
Modify `users` Table:
```php
Schema::table('users', function (Blueprint $table) {
    $table->foreignId('country_id')->constrained()->onDelete('cascade');
});
```

### 📝 Model
#### `Country.php`
```php
public function posts() {
    return $this->hasManyThrough(Post::class, User::class);
}
```

### 📌 Controller Usage
```php
$country = Country::with('posts')->find($id);
$posts = $country->posts;
```

---

## 7. Polymorphic One to Many (`morphMany`)
### A comment can belong to either a post or a video.

### 🛠 Table Structure
#### `comments` Table
```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('comment');
    $table->morphs('commentable'); // Creates `commentable_id` and `commentable_type`
    $table->timestamps();
});
```

### 📝 Model
#### `Comment.php`
```php
public function commentable() {
    return $this->morphTo();
}
```

#### `Post.php`
```php
public function comments() {
    return $this->morphMany(Comment::class, 'commentable');
}
```

#### `Video.php`
```php
public function comments() {
    return $this->morphMany(Comment::class, 'commentable');
}
```

### 📌 Controller Usage
```php
$post = Post::with('comments')->find($id);
$comments = $post->comments;
```

---

## 8. Polymorphic Many to Many (`morphToMany`)
### A tag can belong to both posts and videos.

### 🛠 Table Structure
#### `tags` Table
```php
Schema::create('tags', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->timestamps();
});
```

#### `taggables` Pivot Table
```php
Schema::create('taggables', function (Blueprint $table) {
    $table->id();
    $table->foreignId('tag_id')->constrained()->onDelete('cascade');
    $table->morphs('taggable'); // Creates `taggable_id` and `taggable_type`
    $table->timestamps();
});
```

### 📝 Model
#### `Tag.php`
```php
public function taggables() {
    return $this->morphToMany(Taggable::class, 'taggable');
}
```

#### `Post.php`
```php
public function tags() {
    return $this->morphToMany(Tag::class, 'taggable');
}
```

#### `Video.php`
```php
public function tags() {
    return $this->morphToMany(Tag::class, 'taggable');
}
```

### 📌 Controller Usage
```php
$post = Post::with('tags')->find($id);
$tags = $post->tags;
```


## Eloquent Relationship Summary Table

| Relationship Type                      | Table Name(s)               | Foreign Keys / Special Columns                                  |
|----------------------------------------|-----------------------------|-----------------------------------------------------------------|
| **One to One** (`hasOne`)              | users, profiles             | profiles.user_id                                               |
| **One to Many** (`hasMany`)            | posts, comments             | comments.post_id                                               |
| **Many to One** (`belongsTo`)          | comments, posts             | comments.post_id                                               |
| **Many to Many** (`belongsToMany`)     | users, roles, role_user     | role_user.user_id, role_user.role_id                           |
| **Has One Through** (`hasOneThrough`)  | users, profiles, countries  | profiles.user_id, profiles.country_id                          |
| **Has Many Through** (`hasManyThrough`)| countries, users, posts     | users.country_id, posts.user_id                                |
| **Polymorphic One to Many** (`morphMany`)| comments                   | comments.commentable_id, comments.commentable_type            |
| **Polymorphic Many to Many** (`morphToMany`)| tags, taggables          | taggables.tag_id, taggables.taggable_id, taggables.taggable_type |
