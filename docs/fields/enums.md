Enums
=====

Custom enums can be defined using class constants:

### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\Enum;
use Dms\Core\Model\Object\PropertyTypeDefiner;

class Colour extends Enum
{
    const RED = 'red';
    const GREEN = 'green';
    const BLUE = 'blue';
    
    /**
     * Defines the type of the options contained within the enum.
     *
     * @param PropertyTypeDefiner $values
     *
     * @return void
     */
    protected function defineEnumValues(PropertyTypeDefiner $values)
    {
        $values->asString();
    }
    
    // Static factory methods...

    public static function red() : self
    {
        return new self(self::RED);
    }

    public static function green() : self
    {
        return new self(self::GREEN);
    }

    public static function blue() : self
    {
        return new self(self::BLUE);
    }
}
```

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Car extends Entity
{
    const COLOUR = 'colour';
    const AGE = 'age';
    const WEIGHT = 'weight';
    const HAPPY = 'happy';

    /**
     * @var Colour
     */
    public $colour;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->colour)->asObject(Colour::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Car;


/**
 * The App\Domain\Entities\Car entity mapper.
 */
class CarMapper extends EntityMapper
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
        $map->type(Car::class);
        $map->toTable('cars');

        $map->idToPrimaryKey('id');

        $map->enum(Car::COLOUR)->to('colour')->usingValuesFromConstants();
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
use App\Domain\Services\Persistence\ICarRepository;
use App\Domain\Entities\Car;
use Dms\Common\Structure\Field;
use App\Domain\Entities\Colour;

/**
 * The car module.
 */
class CarModule extends CrudModule
{
    public function __construct(ICarRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('car');

        $module->labelObjects()->fromProperty(Car::ID);


        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('colour', 'Colour')->enum(Colour::class, [
                        Colour::RED => 'Red',
                        Colour::GREEN => 'Green',
                        Colour::BLUE => 'Blue',
                    ])->required()
                )->bindToProperty(Car::COLOUR),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Car::COLOUR)->to(Field::create('colour', 'Colour')->enum(Colour::class, [
                Colour::RED => 'Red',
                Colour::GREEN => 'Green',
                Colour::BLUE => 'Blue',
            ])->required());


            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```