---
title: Logging roles events
weight: 1
---

You can log events for model roles with `Spatie\Permission\Traits\HasRoles` trait being present. Start by configuring the required model to use `HasRoles` trait. This time, it will have to be configured to override the default `roles` relationship method to start using a custom `MorphPivot` model. Following example imports initially public `roles` renamng it to `traitRoles` and making it private.

```php
use HasRoles
{
    roles as private traitRoles;
}
```

Next step would be to define the now missing `roles` method with exactly the same signarure as the original one. This one will just use the previously imported `traitRoles` with only adding a custom pivot model to it. The model could have any name and `UserRole` was used just for convenience and consistency.

```php
public function roles(): BelongsToMany
{
    return $this->traitRoles()->using(UserRole::class);
}
```

Now would be the right moment to create the still missing pivot model and configure it to use logging. Following example shown a fully functonal model with basic logging being configured.

```php
use Illuminate\Database\Eloquent\Relations\MorphPivot;
use Spatie\Activitylog\Traits\LogsActivity;

final class UserRole extends MorphPivot
{
    use LogsActivity;

    protected static $logAttributes = ['*'];
}
```

This would be enough to get the model logging for roles running with the only issue being that this model does not have a unique identifier present. This way a column for identifier will be left empty.

If identifiers are required either for consistency or any other purpose, then this newly created pivot model would need to have a unique identifier present. Start by defining an incrementing identifier for the model as shown below.

```php
public $incrementing = true;
```

The last step would be to add the incrementing unique identifier to the morph relation table by creating a database migration as it is shown in the following example.

```php
class UpdatePermissionTables extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        $tableNames = config('permission.table_names');

        if (empty($tableNames)) {
            throw new \Exception('Error: config/permission.php not loaded. Run [php artisan config:clear] and try again.');
        }

        Schema::table($tableNames['model_has_roles'], function (Blueprint $table) {
            // XXX This seems to already have primary key defined and could raise an error
            $table->bigIncrements('id')->before('role_id');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        $tableNames = config('permission.table_names');

        if (empty($tableNames)) {
            throw new \Exception('Error: config/permission.php not loaded. Run [php artisan config:clear] and try again.');
        }

        Schema::table($tableNames['model_has_roles'], function (Blueprint $table) {
            $table->dropColumn('id');
        });
    }
}
```

Similar approach could be applied to direct permissions, if those are applied to the model with `HasRoles` trait present.
