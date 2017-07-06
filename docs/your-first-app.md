Your first app
==============

To illustrate how to build apps using the DMS, let's go through building a simple TODO
list app. [See installation guide if necessary](./getting-started.md)

Defining your domain model
--------------------------

Firstly lets build our entities. Lets create a file `app/Domain/Entities/TodoItem.php`.

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

/**
 * Your first entity.
 * 
 * An item on the TODO list.
 */
class TodoItem extends Entity
{
    // Define constants for property names
    const DESCRIPTION = 'description';
    const COMPLETED = 'completed';
    
    /**
     * @var string 
     */
    public $description;
    
    /**
     * @var bool 
     */
    public $completed;
    
    /**
     * Initialises a new TODO item.
     * 
     * @param string $description
     */
    public function __construct(string $description)
    {
        parent::__construct();
        $this->description = $description;
        $this->completed   = false;
    }
    
    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        // Enables strong typing for this entity
        $class->property($this->description)->asString();
        
        $class->property($this->completed)->asBool();
    }
}
```

Building your persistence layer
-------------------------------

So now we have our entities, we need a way to persist them to a database.

### Defining the mapper

Create the entity mapper under `app/Infrastructure/Persistence/TodoItemMapper.php`.
Configure how to map the entity to the database table.

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\TodoItem;

/**
 * Your first entity mapper.
 * 
 * The App\Domain\Entities\TodoItem entity mapper.
 */
class TodoItemMapper extends EntityMapper
{
    /**
     * Defines the entity mapper
     *
     * @param MapperDefinition $map
     *
     * @return void
     */
    protected function define(MapperDefinition $map)
    {
        $map->type(TodoItem::class);
        $map->toTable('todo_items');

        $map->idToPrimaryKey('id');

        $map->property(TodoItem::DESCRIPTION)->to('description')->asVarchar(255);

        $map->property(TodoItem::COMPLETED)->to('completed')->asBool();
    }
}
```

Register the mapper in `app/AppOrm.php`:

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

### Defining the repository interface

Create the repository interface `app/Domain/Services/ITodoItemRepository.php`.

```php
<?php declare(strict_types = 1);

namespace App\Domain\Services\Persistence;

use Dms\Core\Model\ICriteria;
use Dms\Core\Model\ISpecification;
use Dms\Core\Persistence\IRepository;
use App\Domain\Entities\TodoItem;

/**
 * The repository for the App\Domain\Entities\TodoItem entity.
 */
interface ITodoItemRepository extends IRepository
{
    /**
     * {@inheritDoc}
     *
     * @return TodoItem[]
     */
    public function getAll() : array;

    /**
     * {@inheritDoc}
     *
     * @return TodoItem
     */
    public function get($id);

    /**
     * {@inheritDoc}
     *
     * @return TodoItem[]
     */
    public function getAllById(array $ids) : array;

    /**
     * {@inheritDoc}
     *
     * @return TodoItem|null
     */
    public function tryGet($id);

    /**
     * {@inheritDoc}
     *
     * @return TodoItem[]
     */
    public function tryGetAll(array $ids) : array;

    /**
     * {@inheritDoc}
     *
     * @return TodoItem[]
     */
    public function matching(ICriteria $criteria) : array;

    /**
     * {@inheritDoc}
     *
     * @return TodoItem[]
     */
    public function satisfying(ISpecification $specification) : array;
}
```

### Defining the repository implementation

Create the repository implementation `app/Infrastructure/Persistence/DbTodoItemRepository.php`.

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Connection\IConnection;
use Dms\Core\Persistence\Db\Mapping\IOrm;
use Dms\Core\Persistence\DbRepository;
use App\Domain\Services\Persistence\ITodoItemRepository;
use App\Domain\Entities\TodoItem;

/**
 * The database repository implementation for the App\Domain\Entities\TodoItem entity.
 */
class DbTodoItemRepository extends DbRepository implements ITodoItemRepository
{
    public function __construct(IConnection $connection, IOrm $orm)
    {
        parent::__construct($connection, $orm->getEntityMapper(TodoItem::class));
    }
}
```

Bind your repository implementation to the interface in `app/Providers/AppServiceProvider.php`

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

### Migrating the database

To sync the database we must generate a migration and run it:

```
 # Auto-generate a migration to sync the database
 php artisan dms:make:migration add_todo_items_table
 # Run the migration
 php artisan migrate
```

You can review the generated migration under the `database/migrations/` directory

Building the CMS
----------------

### Defining your modules

To build the backend we must first configure the module `app/Cms/Modules/TodoItemModule.php`

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\ITodoItemRepository;
use App\Domain\Entities\TodoItem;
use Dms\Common\Structure\Field;

/**
 * The todo-item module.
 */
class TodoItemModule extends CrudModule
{
    public function __construct(ITodoItemRepository $dataSource, IAuthSystem $authSystem)
    {
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('todo-item');

        $module->labelObjects()->fromProperty(TodoItem::DESCRIPTION);

        $module->metadata([
            'icon' => 'list', // Choose icon from http://fontawesome.io/icons/
        ]);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('description', 'Description')->string()->required()
                )->bindToProperty(TodoItem::DESCRIPTION),
                //
                $form->field(
                    Field::create('completed', 'Completed')->bool()
                )->bindToProperty(TodoItem::COMPLETED),
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(TodoItem::DESCRIPTION)->to(Field::create('description', 'Description')->string()->required());
            $table->mapProperty(TodoItem::COMPLETED)->to(Field::create('completed', 'Completed')->bool());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```

### Defining your package

Create the package (a group of modules) `app/Cms/TodoAppPackage.php`

```php
<?php declare(strict_types = 1);

namespace App\Cms;

use Dms\Core\Package\Definition\PackageDefinition;
use Dms\Core\Package\Package;
use App\Cms\Modules\TodoItemModule;

/**
 * The todo-app package.
 */
class TodoAppPackage extends Package
{
    /**
     * Defines the structure of this cms package.
     *
     * @param PackageDefinition $package
     *
     * @return void
     */
    protected function define(PackageDefinition $package)
    {
        $package->name('todo-app');

        $package->metadata([
            'icon' => 'check-square', // Choose icon from http://fontawesome.io/icons/
        ]);

        $package->modules([
            'todo-item' => TodoItemModule::class,
        ]);
    }
}
```

Register the package in `app/AppCms.php`

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

Visit `http://your-app-domain/dms` and login to see your first backend:

![Todo Module](/resources/images/cms/todo-module-1.jpg)