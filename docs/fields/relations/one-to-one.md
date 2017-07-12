One to One
==========

A one to one bi-directional relationship between entities can be defined as follows.

### Entity Structure

#### app/Domain/Entities/Country.php

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Country extends Entity
{
    const NAME = 'name';
    const CAPITAL_CITY = 'capitalCity';

    /**
     * @var string
     */
    public $name;

    /**
     * @var CapitalCity
     */
    public $capitalCity;

    /**
     * Country constructor.
     *
     * @param string      $name
     * @param CapitalCity $capitalCity
     */
    public function __construct(string $name, CapitalCity $capitalCity)
    {
        parent::__construct();
        $this->name           = $name;
        $this->capitalCity    = $capitalCity;
        $capitalCity->country = $this;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->capitalCity)->asObject(CapitalCity::class);
    }
}
```

#### app/Domain/Entities/CapitalCity.php

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class CapitalCity extends Entity
{
    const NAME = 'name';
    const COUNTRY = 'country';

    /**
     * @var string
     */
    public $name;

    /**
     * @var Country
     */
    public $country;

    /**
     * CapitalCity constructor.
     *
     * @param string  $name
     * @param Country $country
     */
    public function __construct(string $name, Country $country)
    {
        parent::__construct();
        $this->name           = $name;
        $this->country        = $country;
        $country->capitalCity = $this;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->country)->asObject(Country::class);
    }
}
```


### Mapper Configuration

#### app/Infrastructure/Persistence/CountryMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Country;
use App\Domain\Entities\CapitalCity;

/**
 * The App\Domain\Entities\Country entity mapper.
 */
class CountryMapper extends EntityMapper
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
        $map->type(Country::class);
        $map->toTable('countries');

        $map->idToPrimaryKey('id');

        $map->property(Country::NAME)->to('name')->asVarchar(255);

        $map->column('capital_city_id')->asUnsignedInt();
        $map->relation(Country::CAPITAL_CITY)
            ->to(CapitalCity::class)
            ->manyToOne()
            ->withBidirectionalRelation(CapitalCity::COUNTRY)
            ->withRelatedIdAs('capital_city_id');
    }
}
```

#### app/Infrastructure/Persistence/CapitalCityMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\CapitalCity;
use App\Domain\Entities\Country;

/**
 * The App\Domain\Entities\CapitalCity entity mapper.
 */
class CapitalCityMapper extends EntityMapper
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
        $map->type(CapitalCity::class);
        $map->toTable('capital_cities');

        $map->idToPrimaryKey('id');

        $map->property(CapitalCity::NAME)->to('name')->asVarchar(255);

        $map->relation(CapitalCity::COUNTRY)
            ->to(Country::class)
            ->toOne()
            ->identifying()
            ->withBidirectionalRelation(Country::CAPITAL_CITY)
            ->withParentIdAs('capital_city_id');
    }
}
```

### Module Configuration

#### app/Cms/Modules/CountryModule.php

```php
<?php declare(strict_types=1);

namespace App\Cms\Modules;

use App\Domain\Entities\CapitalCity;
use App\Domain\Entities\Country;
use App\Domain\Services\Persistence\ICapitalCityRepository;
use App\Domain\Services\Persistence\ICountryRepository;
use Dms\Common\Structure\Field;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;

/**
 * The country module.
 */
class CountryModule extends CrudModule
{
    /**
     * @var ICapitalCityRepository
     */
    protected $capitalCityRepository;

    public function __construct(ICountryRepository $dataSource, IAuthSystem $authSystem, ICapitalCityRepository $capitalCityRepository)
    {
        $this->capitalCityRepository = $capitalCityRepository;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('country');

        $module->labelObjects()->fromProperty(Country::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(Country::NAME),
                //
                //
            ]);

            if ($form->isCreateForm()) {
                $form->continueSection([
                    $form->field(
                        Field::create('capital_city', 'Capital City')->string()->required()
                    )->bindToCallbacks(
                        function () {
                            // Unused
                        },
                        function (Country $country, string $cityName) {
                            $country->capitalCity = new CapitalCity($cityName, $country);
                        }
                    ),
                ]);
            } else {
                $form->continueSection([
                    $form->field(
                        Field::create('capital_city', 'Capital City')
                            ->entityFrom($this->capitalCityRepository)
                            ->required()
                            ->labelledBy(CapitalCity::NAME)
                            // Add this line if you want to load the
                            // options asynchronously via autocomplete
                            ->searchableBy(Country::NAME)
                    )->bindToProperty(Country::CAPITAL_CITY),
                ]);
            }
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Country::NAME)->to(Field::create('name', 'Name')->string()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```

#### app/Cms/Modules/CapitalCityModule.php

```php
<?php declare(strict_types=1);

namespace App\Cms\Modules;

use App\Domain\Entities\CapitalCity;
use App\Domain\Entities\Country;
use App\Domain\Services\Persistence\ICapitalCityRepository;
use App\Domain\Services\Persistence\ICountryRepository;
use Dms\Common\Structure\Field;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;

/**
 * The capital-city module.
 */
class CapitalCityModule extends CrudModule
{
    /**
     * @var ICountryRepository
     */
    protected $countryRepository;

    public function __construct(ICapitalCityRepository $dataSource, IAuthSystem $authSystem, ICountryRepository $countryRepository)
    {
        $this->countryRepository = $countryRepository;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('capital-city');

        $module->labelObjects()->fromProperty(CapitalCity::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(CapitalCity::NAME),
            ]);

            if ($form->isCreateForm()) {
                $form->continueSection([
                    $form->field(
                        Field::create('country', 'Country')->string()->required()
                    )->bindToCallbacks(
                        function () {
                             // Unused
                        },
                        function (CapitalCity $capitalCity, string $countryName) {
                            $capitalCity->country = new Country($countryName, $capitalCity);
                        }
                    )
                ]);
            } else {
                $form->continueSection([
                    $form->field(
                        Field::create('country', 'Country')
                            ->entityFrom($this->countryRepository)
                            ->required()
                            ->labelledBy(Country::NAME)
                            // Add this line if you want to load the
                            // options asynchronously via autocomplete
                            ->searchableBy(Country::NAME)
                    )->bindToProperty(CapitalCity::COUNTRY),
                ]);
            }
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(CapitalCity::NAME)->to(Field::create('name', 'Name')->string()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```