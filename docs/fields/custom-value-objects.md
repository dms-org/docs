Custom Value Objects
====================

You may find yourself repeating a bunch of fields throughout your application, in this case you can
extract these fields into your own custom value objects which can then be reused throughout your app.

### Entity Structure

#### app/Domain/Entities/Engine.php

Your custom value object class

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\ValueObject;

class Engine extends ValueObject
{
    const NAME = 'name';
    const HORSE_POWER = 'horsePower';

    /**
     * @var string
     */
    public $name;

    /**
     * @var int
     */
    public $horsePower;

    /**
     * Defines the structure of this class.
     *
     * @param ClassDefinition $class
     */
    protected function define(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->horsePower)->asInt();
    }
}
```

#### app/Domain/Entities/Vehicle.php

An entity which contains your value object as a field.

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Vehicle extends Entity
{
    const ENGINE = 'engine';

    /**
     * @var Engine
     */
    public $engine;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->engine)->asObject(Engine::class);
    }
}
```



### Mapper Configuration

#### app/Infrastructure/Persistence/EngineMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\IndependentValueObjectMapper;
use App\Domain\Entities\Engine;

/**
 * The App\Domain\Entities\Engine value object mapper.
 */
class EngineMapper extends IndependentValueObjectMapper
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
        $map->type(Engine::class);

        $map->property(Engine::NAME)->to('name')->asVarchar(255);

        $map->property(Engine::HORSE_POWER)->to('horse_power')->asInt();
    }
}
```

#### app/Infrastructure/Persistence/VehicleMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Vehicle;
use App\Infrastructure\Persistence\EngineMapper;

/**
 * The App\Domain\Entities\Vehicle entity mapper.
 */
class VehicleMapper extends EntityMapper
{
    /**
     * Defines the value object mapper
     *
     * @param MapperDefinition $map
     *
     * @return void
     */
    protected function define(MapperDefinition $map)
    {
        $map->type(Vehicle::class);
        $map->toTable('vehicles');

        $map->idToPrimaryKey('id');

        // Embeds the columns of the value object in the 'vehicles' table
        // Each column will be prefixed so they will become 'engine_name' and 'engine_horse_power'
        $map->embedded(Vehicle::ENGINE)
            ->withColumnsPrefixedBy('engine_')
            ->using(new EngineMapper());
    }
}
```

### Module Configuration

#### app/Cms/Modules/Fields/EngineField.php

You can define custom fields for value objects

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules\Fields;

use Dms\Core\Common\Crud\Definition\Form\ValueObjectFieldDefinition;
use Dms\Core\Common\Crud\Form\ValueObjectField;
use App\Domain\Entities\Engine;
use Dms\Common\Structure\Field;

/**
 * The App\Domain\Entities\Engine value object field.
 */
class EngineField extends ValueObjectField
{
    public function __construct(string $name, string $label)
    {
        parent::__construct($name, $label);
    }

    /**
     * Defines the structure of this value object field.
     *
     * @param ValueObjectFieldDefinition $form
     *
     * @return void
     */
    protected function define(ValueObjectFieldDefinition $form)
    {
        $form->bindTo(Engine::class);

        $form->section('Details', [
            $form->field(
                Field::create('name', 'Name')->string()->required()
            )->bindToProperty(Engine::NAME),
            //
            $form->field(
                Field::create('horse_power', 'Horse Power')->int()->required()
            )->bindToProperty(Engine::HORSE_POWER),
            //
        ]);
    }
}
```
#### app/Cms/Modules/VehicleModule.php

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IVehicleRepository;
use App\Domain\Entities\Vehicle;
use App\Cms\Modules\Fields\EngineField;

/**
 * The vehicle module.
 */
class VehicleModule extends CrudModule
{
    public function __construct(IVehicleRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('vehicle');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    // Reference your custom value object field
                    (new EngineField('engine', 'Engine'))->required()
                )->bindToProperty(Vehicle::ENGINE),
                //
            ]);
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Vehicle::ENGINE)->to((new EngineField('engine', 'Engine'))->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```