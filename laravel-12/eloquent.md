## Configuring Eloquent Strictness
Add these to AppServiceProvider for sane default eloquent strictness guards:
```php
Model::preventLazyLoading(! $this->app->isProduction());
```
- Lazy loading is prevented, avoid N+1 problem.
- Errors when a model attemps to lazy load a relationship.
- Must explicitly eager load a relationship using `with()`.
```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```
- Prevents Laravel from ignoring attributes that don’t exist on the model's `$fillable` or database table.
- Errors if you try to set or retrieve an attribute that isn’t actually defined for that model.
- Catch typos, outdated column names, or incorrect mass assignment keys during development.

## Retrieving Models
Most of the time use `lazy()` in queries for chunking + memory efficient per model access.
```php
use App\Models\Flight;

foreach (Flight::lazy() as $flight) {
    // ...
}
```

```php
use App\Models\Flight;

Flight::where('departed', true)
    ->lazyById(200, column: 'id')
    ->each->update(['departed' => false]);
```

## Model Events
Prefer Observer to handle actions (event listeners) for specific or all model events.
```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```
