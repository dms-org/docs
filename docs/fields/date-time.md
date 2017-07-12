Date and time
=============

Date and time fields can be modelled using value objects.

 - `Dms\Common\Structure\DateTime\Date` - A date value `eg 2000-10-12`
 - `Dms\Common\Structure\DateTime\TimeOfDay` - A time value `eg 06:12:56`
 - `Dms\Common\Structure\DateTime\DateTime` - A date and time value `eg 2000-10-12 06:12:56`
 - `Dms\Common\Structure\DateTime\TimezonedDateTime` - A date and time in a particular timezone value `eg 2000-10-12 06:12:56 Australia/Melbourne`

These classes wrap a native `DateTime` instance and provide more descriptive API
for each type of date/time class.

### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Common\Structure\DateTime\Date;
use Dms\Common\Structure\DateTime\DateTime;
use Dms\Common\Structure\DateTime\TimeOfDay;
use Dms\Common\Structure\DateTime\TimezonedDateTime;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Event extends Entity
{
    const DATE = 'date';
    const TIME = 'time';
    const DATETIME = 'dateTime';
    const TIMEZONED_DATETIME = 'timezoneDateTime';

    /**
     * @var Date
     */
    public $date;

    /**
     * @var TimeOfDay
     */
    public $time;

    /**
     * @var DateTime
     */
    public $dateTime;

    /**
     * @var TimezonedDateTime
     */
    public $timezoneDateTime;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->date)->asObject(Date::class);

        $class->property($this->time)->asObject(TimeOfDay::class);

        $class->property($this->dateTime)->asObject(DateTime::class);

        $class->property($this->timezoneDateTime)->asObject(TimezonedDateTime::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Event;
use Dms\Common\Structure\DateTime\Persistence\DateMapper;
use Dms\Common\Structure\DateTime\Persistence\TimeOfDayMapper;
use Dms\Common\Structure\DateTime\Persistence\DateTimeMapper;
use Dms\Common\Structure\DateTime\Persistence\TimezonedDateTimeMapper;

/**
 * The App\Domain\Entities\Event entity mapper.
 */
class EventMapper extends EntityMapper
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
        $map->type(Event::class);
        $map->toTable('events');

        $map->idToPrimaryKey('id');

        $map->embedded(Event::DATE)
            ->using(new DateMapper('date'));

        $map->embedded(Event::TIME)
            ->using(new TimeOfDayMapper('time'));

        $map->embedded(Event::DATETIME)
            ->using(new DateTimeMapper('date_time'));

        $map->embedded(Event::TIMEZONED_DATETIME)
            ->using(new TimezonedDateTimeMapper('timezone_date_time_date_time', 'timezone_date_time_timezone'));
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
use App\Domain\Services\Persistence\IEventRepository;
use App\Domain\Entities\Event;
use Dms\Common\Structure\Field;

/**
 * The event module.
 */
class EventModule extends CrudModule
{
    public function __construct(IEventRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('event');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('date', 'Date')->date()->required()
                )->bindToProperty(Event::DATE),
                //
                $form->field(
                    Field::create('time', 'Time')->time()->required()
                )->bindToProperty(Event::TIME),
                //
                $form->field(
                    Field::create('date_time', 'Date Time')->dateTime()->required()
                )->bindToProperty(Event::DATETIME),
                //
                $form->field(
                    Field::create('timezone_date_time', 'Timezone Date Time')->dateTimeWithTimezone()->required()
                )->bindToProperty(Event::TIMEZONED_DATETIME),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Event::DATE)->to(Field::create('date', 'Date')->date()->required());
            $table->mapProperty(Event::TIME)->to(Field::create('time', 'Time')->time()->required());
            $table->mapProperty(Event::DATETIME)->to(Field::create('date_time', 'Date Time')->dateTime()->required());
            $table->mapProperty(Event::TIMEZONED_DATETIME)->to(Field::create('timezone_date_time', 'Timezone Date Time')->dateTimeWithTimezone()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```