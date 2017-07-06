Scaffolding
===========

For each and every project, setting up your entities, mappers, modules can feel
repetitive and painful. The DMS provides powerful scaffolding commands that
allow you to generate mappers for the ORM and modules for the CMS out of the box.

Scaffolding the ORM
-------------------

```
php artisan dms:scaffold:persistence
```

Running this command will look for entities in the `App\Domain\Entities` namespace
and auto-generate your persistence layer.

 - Mapper and repository classes will be put under `app/Infrastructure/Pesistence`
 - Repository interfaces will be put under `app/Domain/Services/Pesistence`

All that is left to do is register your mappers in `app/AppOrm.php`


```php
<?php declare(strict_types = 1);

namespace App;

use App\Domain\Entities\TodoItem;
use App\Infrastructure\Persistence\TodoItemMapper;
use Dms\Core\Persistence\Db\Mapping\Definition\Orm\OrmDefinition;
use Dms\Core\Persistence\Db\Mapping\Orm;
use Dms\Web\Laravel\Persistence\Db\DmsOrm;

/**
 * The application's orm.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppOrm extends Orm
{
    /**
     * Defines the object mappers registered in the orm.
     *
     * @param OrmDefinition $orm
     *
     * @return void
     */
    protected function define(OrmDefinition $orm)
    {
        $orm->enableLazyLoading();

        $orm->encompass(DmsOrm::inDefaultNamespace());

        // You can register your mappers here
        $orm->entities([
            TodoItem::class => TodoItemMapper::class
        ]);
    }
}
```

And to bind your repository interfaces to the implementations


```php
<?php

namespace App\Providers;

use App\AppCms;
use App\AppOrm;
use App\Domain\Services\Persistence\ITodoItemRepository;
use App\Infrastructure\Persistence\DbTodoItemRepository;
use Dms\Core\ICms;
use Dms\Core\Persistence\Db\Mapping\IOrm;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(IOrm::class, AppOrm::class);
        $this->app->singleton(ICms::class, AppCms::class);

        // Bind your repositories here
        $this->app->singleton(ITodoItemRepository::class, DbTodoItemRepository::class);
    }
}
```

Generating Migrations
---------------------

```
php artisan dms:make:migration your_migration_name
```

Running this command will look at the current structure of your mappers
and the current database structure and generate a migration class to sync
your database with the current entity structure.

 - A migration will be generated under `database/migrations`

An example auto-generated migration:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class AddTodoItemsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('todo_items', function (Blueprint $table) {
            $table->integer('id')->autoIncrement()->unsigned();
            $table->string('description', 255);
            $table->boolean('completed');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('todo_items');
    }
}
```

*This command only generates the migration, you will have to run
 `php artisan migrate` to actually run the migration*

*It is recommended to review the migrations to ensure the database is synced
expected*

Scaffolding the CMS
-------------------

```
php artisan dms:scaffold:cms your-app-name
```

Running this command will look for entities in the `App\Domain\Entities` namespace
and auto-generate your CMS layer.

 - Module classes will be put under `app/Cms/Modules`
 - A package class will be put under `app/Cms/YouAppNamePackage.php`

All that is left to do is register your package in `app/AppCms.php`

```php
<?php declare(strict_types=1);

namespace App;

use App\Cms\TodoAppPackage;
use Dms\Core\Cms;
use Dms\Core\CmsDefinition;
use Dms\Package\Analytics\AnalyticsPackage;
use Dms\Web\Laravel\Auth\AdminPackage;
use Dms\Web\Laravel\Document\PublicFilePackage;

/**
 * The application's cms.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppCms extends Cms
{
    /**
     * Defines the structure and installed packages of the cms.
     *
     * @param CmsDefinition $cms
     *
     * @return void
     */
    protected function define(CmsDefinition $cms)
    {
        $cms->packages([
            // Default packages installed out of the box
            'admin'     => AdminPackage::class,
            'documents' => PublicFilePackage::class,
            'analytics' => AnalyticsPackage::class,

            // Register your packages here...
            'todo-app' => TodoAppPackage::class,
        ]);
    }
}
```