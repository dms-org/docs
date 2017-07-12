Location
========

Geographical locations can be modelled using value objects:
 
 - `Dms\Common\Structure\Geo\LatLng` - A latitude/longitude coordinate
 - `Dms\Common\Structure\Geo\StreetAddress` - A street address string
 - `Dms\Common\Structure\Geo\StreetAddressWithLatLng` - A street address string and its associated lat/lng coordinates
 
 
### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Common\Structure\Geo\LatLng;
use Dms\Common\Structure\Geo\StreetAddress;
use Dms\Common\Structure\Geo\StreetAddressWithLatLng;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Building extends Entity
{
    const ADDRESS = 'address';
    const LAT_LNG = 'latLng';
    const ADDRESS_WITH_LAT_LNG = 'addressWithLatLng';

    /**
     * @var StreetAddress
     */
    public $address;

    /**
     * @var LatLng
     */
    public $latLng;

    /**
     * @var StreetAddressWithLatLng
     */
    public $addressWithLatLng;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->address)->asObject(StreetAddress::class);

        $class->property($this->latLng)->asObject(LatLng::class);
        
        $class->property($this->addressWithLatLng)->asObject(StreetAddressWithLatLng::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Building;
use Dms\Common\Structure\Geo\Persistence\StreetAddressMapper;
use Dms\Common\Structure\Geo\Persistence\StreetAddressWithLatLngMapper;
use Dms\Common\Structure\Geo\Persistence\LatLngMapper;

/**
 * The App\Domain\Entities\Building entity mapper.
 */
class BuildingMapper extends EntityMapper
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
        $map->type(Building::class);
        $map->toTable('buildings');

        $map->idToPrimaryKey('id');

        $map->embedded(Building::ADDRESS)
            ->using(new StreetAddressMapper('address'));

        $map->embedded(Building::LAT_LNG)
            ->using(new LatLngMapper('lat', 'lng'));

        $map->embedded(Building::ADDRESS_WITH_LAT_LNG)
            ->using(new StreetAddressWithLatLngMapper('address1', 'lat1', 'lng1'));
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
use App\Domain\Services\Persistence\IBuildingRepository;
use App\Domain\Entities\Building;
use Dms\Common\Structure\Field;

/**
 * The building module.
 */
class BuildingModule extends CrudModule
{
    public function __construct(IBuildingRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('building');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('address', 'Address')->streetAddress()->required()
                )->bindToProperty(Building::ADDRESS),
                //
                $form->field(
                    Field::create('lat_lng', 'Lat Lng')->latLng()->required()
                )->bindToProperty(Building::LAT_LNG),
                //
                $form->field(
                    Field::create('address_with_lat_lng', 'Address With Lat Lng')->streetAddressWithLatLng()->required()
                )->bindToProperty(Building::ADDRESS_WITH_LAT_LNG),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Building::ADDRESS)->to(Field::create('address', 'Address')->streetAddress()->required());
            $table->mapProperty(Building::LAT_LNG)->to(Field::create('lat_lng', 'Lat Lng')->latLng()->required());
            $table->mapProperty(Building::ADDRESS_WITH_LAT_LNG)->to(Field::create('address_with_lat_lng', 'Address With Lat Lng')->streetAddressWithLatLng()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```