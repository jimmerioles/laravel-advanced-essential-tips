## One to Many / Has Many
Always use `chaperone()` to avoid sneaky N + 1.
```php
<?php
 
...
 
class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```
- Normally, even with eager loading, accessing the parent from each child (e.g. `$comment->post`) can trigger additional queries per child. Using `->chaperone()` hydrates the parent model automatically, preventing that sneaky N + 1 problem.

### One to Many (Inverse) / Belongs To
Use `whereBelongsTo()` to get all related children of a parent or collection of parent model.
```php
// Get all posts that belongs to $user
$posts = Post::whereBelongsTo($user)->get();

// Get all posts that belongs to vip $users
$users = User::where('vip', true)->get();
$posts = Post::whereBelongsTo($users)->get();
```
## Has One of Many :rocket:
If you’ve ever written a query that says “get the latest X per Y” and you want it to eager load efficiently (only 2 queries vs looping n + 1), you’re in hasOneOfMany territory.

You can grab “the one related child record that matters” from a large parent set without hammering the database.

```php
class Customer extends Model
{

    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }

    public function latestOrder(): HasOne
    {
        return $this->orders()->one()->latestOfMany();

        // If hasMany orders() relationship is not already declared: 
        // return $this->hasOne(Order::class)->latestOfMany();
    }
}

$customers = Customer::with('latestOrder')->get();
```
What happens behind the scenes:
- Laravel runs 1 query for all users.
- Laravel runs 1 query that uses a JOIN and grouping to fetch each customer’s latest order in bulk.
- 2 queries, even if you have 5,000 customers.

Example SQL (simplified):
```sql
select * from orders
where id in (
    select max(id) from orders group by customer_id
)
```

## Has One Through
When you need to get one related record through another table for many parents at once, eager loading keeps it at 2 queries total — even if you have thousands of parent records (avoids N + 1).

Example:
- Store → Manager → ContactInfo
- You want all store contact infos in a dashboard.
```php
class Store extends Model
{
    public function manager(): HasOne
    {
        return $this->hasOne(Manager::class);
    }

    public function managerContact(): HasOneThrough
    {
        return $this->hasOneThrough(
            ContactInfo::class, // Final model
            Manager::class,     // Intermediate model
            'store_id',         // FK on managers table
            'manager_id',       // FK on contact_infos table
            'id',               // Local key on stores table
            'id'                // Local key on managers table
        );

        // Or with Laravel's "through" syntax (if relationships are declared):
        // return $this->through('manager')->has('contactInfo');
    }
}

$stores = Store::with('managerContact')->get(); // Always 2 queries
```

## Has Many Through
When you need to get many related records through another table for many parents at once, eager loading keeps it at 2 queries total — even if you have thousands of parent records (avoids N + 1).
- Branch → Employee → Sale
- You want all branch sales in a report/dashboard.
```php
class Branch extends Model
{
    public function employees(): HasMany
    {
        return $this->hasMany(Employee::class);
    }

    public function sales(): HasManyThrough
    {
        return $this->hasManyThrough(
            Sale::class,      // Final model
            Employee::class,  // Intermediate model
            'branch_id',      // FK on employees table
            'employee_id',    // FK on sales table
            'id',             // Local key on branches table
            'id'              // Local key on employees table
        );

        // Or with Laravel's "through" syntax (if relationships are declared):
        // return $this->through('employees')->has('sales');
    }
}

$branches = Branch::with('sales')->get(); // Always 2 queries
```
