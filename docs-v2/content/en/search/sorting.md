---
title: Sort
menuTitle: Sort
description: Sorting 
category: Search & Filters
position: 13
---

 ## Definition

During index requests, we usually have to sort them by some specific attributes. This requires the `$sort` configuration:

```php
class PostRepository extends Repository
{
    public static array $sort = ['id'];
```

Performing a request requires the query param sort:

## Descending sorting

Sorting DESC requires a minus (`-`) sign before the attribute name:

 ```http_request
GET: /api/restify/posts?sort=-id
```

Sorting ASC:

 ```http_request
GET: /api/restify/posts?sort=id
```

or with a plus sign before the field:

 ```http_request
GET: /api/restify/posts?sort=+id
```

## Sort using relation

Sometimes, you may need to sort using a `belongsTo` or `hasOne` relationship. 

This becomes a breeze with Restify. First you have to instruct your sort to use a relationship:

### HasOne sorting
By using a `related` relationship, it becomes very easy to define a sortable by having a related one.

You simply add the `->sortable()` method to the relationship:

```php
// UserRepository.php

public function related(): array
{
    return [
        'post' => HasOne::make('post', PostRepository::class)->sortable('title'),
    ];
}
```

<alert>

The `sortable` method accepts the column (or fully qualified column name) of the related model.

</alert>


The API request will always have to use the full path to the `attributes`:

```bash
GET: /api/restify/posts?sort=post.attributes.title
```

The structure of the `sort` query param value always includes these 3 parts:

- `post` - the name of the relation defined in the `related` method
- `attributes` - a generic json:api term
- `title` - the column name from the database of the related model

### BelongsTo sorting

The belongsTo sorting works in a similar way.

You simply add the `->sortable()` method to the relationship:

```php
// PostRepository.php

public function related(): array
{
    return [
        'user' => BelongsTo::make('user', UserRepository::class)->sortable('name'),
    ];
}
```

### Using custom sortable filter

You can override the `sorts` method and return an instance of `SortableFilter` that might be instructed to use a relationship:

```php
// PostRepository
use Binaryk\LaravelRestify\Fields\BelongsTo;
use Binaryk\LaravelRestify\Filters\SortableFilter;

public static function sorts(): array
{
    return [
        'users.name' => SortableFilter::make()
            ->setColumn('users.name')
            ->usingRelation(
                BelongsTo::make('user', 'user', UserRepository::class),
        )
    ];
}
```

Make sure that the column is fully qualified (including the table name).

The request could look like this:

```http request
GET: /api/restify/posts?sort=-users.name
```

This will return all posts sorted descendingly by users' names.

<alert type="info">

As you can notice, we have typed twice the `users.name` (on the array key, and as an argument in the `setColumn` method). As soon as you use the fully qualified key name, you can avoid the `setColumn` call since the column will be injected automatically based on the `sorts` key.

</alert>

## Sort using closure

If you have a quick sort method, you can use a closure to sort your data:

```php
// PostRepository
use Binaryk\LaravelRestify\Http\Requests\RestifyRequest;

public static function sorts(): array
{
    return [
        'users.name' => function(RestifyRequest $request, $query, $direction) {
            // custom sort
        }
    ];
}
```

## Get available sorts

You can use the following request to get sortable attributes for a repository:

```http request
/api/restify/posts/filters?only=sortables
```

<alert type="info">

To get all the filters, you can use `/api/restify/posts/filters?only=sortables,matches,searchables`.
 
</alert>
