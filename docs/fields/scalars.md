Scalars
=======

Native data types (strings, integers, floats, booleans) can be expressed as follows

### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Person extends Entity
{
    const NAME = 'name';
    const AGE = 'age';
    const WEIGHT = 'weight';
    const HAPPY = 'happy';

    /**
     * @var string
     */
    public $name;

    /**
     * @var int
     */
    public $age;

    /**
     * @var float
     */
    public $weight;

    /**
     * @var bool
     */
    public $happy;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->age)->asInt();

        $class->property($this->weight)->asFloat();

        $class->property($this->happy)->asBool();
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Person;


/**
 * The App\Domain\Entities\Person entity mapper.
 */
class PersonMapper extends EntityMapper
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
        $map->type(Person::class);
        $map->toTable('people');

        $map->idToPrimaryKey('id');

        $map->property(Person::NAME)->to('name')->asVarchar(255);

        $map->property(Person::AGE)->to('age')->asInt();

        $map->property(Person::WEIGHT)->to('weight')->asDecimal(16, 8);

        $map->property(Person::HAPPY)->to('happy')->asBool();
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
use App\Domain\Services\Persistence\IPersonRepository;
use App\Domain\Entities\Person;
use Dms\Common\Structure\Field;

/**
 * The person module.
 */
class PersonModule extends CrudModule
{
    public function __construct(IPersonRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('person');

        $module->labelObjects()->fromProperty(Person::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(Person::NAME),
                //
                $form->field(
                    Field::create('age', 'Age')->int()->required()
                )->bindToProperty(Person::AGE),
                //
                $form->field(
                    Field::create('weight', 'Weight')->decimal()->required()
                )->bindToProperty(Person::WEIGHT),
                //
                $form->field(
                    Field::create('happy', 'Happy')->bool()
                )->bindToProperty(Person::HAPPY),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Person::NAME)->to(Field::create('name', 'Name')->string()->required());
            $table->mapProperty(Person::AGE)->to(Field::create('age', 'Age')->int()->required());
            $table->mapProperty(Person::WEIGHT)->to(Field::create('weight', 'Weight')->decimal()->required());
            $table->mapProperty(Person::HAPPY)->to(Field::create('happy', 'Happy')->bool());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```