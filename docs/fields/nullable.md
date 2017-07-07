Nullable
========

Fields can be defined as nullable which means they can possible contain a `null` value.
This is useful for optional fields.

### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class User extends Entity
{
    const NAME = 'name';

    /**
     * @var string|null
     */
    public $name;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->nullable()->asString();
    }

    /**
     * @return bool
     */
    public function isAnonymous() : bool
    {
        return $this->name === null;
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\User;


/**
 * The App\Domain\Entities\User entity mapper.
 */
class UserMapper extends EntityMapper
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
        $map->type(User::class);
        $map->toTable('users');

        $map->idToPrimaryKey('id');

        $map->property(User::NAME)->to('name')->nullable()->asVarchar(255);
    }
}
```

### Module Configuration

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IUserRepository;
use App\Domain\Entities\User;
use Dms\Common\Structure\Field;

/**
 * The user module.
 */
class UserModule extends CrudModule
{
    public function __construct(IUserRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('user');

        $module->labelObjects()->fromProperty(User::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    // Omit ->required() to make the field optional
                    Field::create('name', 'Name')->string()
                )->bindToProperty(User::NAME),
                //
            ]);
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(User::NAME)->to(Field::create('name', 'Name')->string());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```