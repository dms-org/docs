Date and time ranges
====================

Date and time range fields can be modelled using value objects.

 - `Dms\Common\Structure\DateTime\DateRange` - A date range `eg 2000-10-12 until 2002-03-05`
 - `Dms\Common\Structure\DateTime\TimeRage` - A time range value `eg 06:12:56 until 14:40:23`
 - `Dms\Common\Structure\DateTime\DateTimeRange` - A date and time range value `eg 2000-10-12 06:12:56 until 2002-03-05 14:40:23`
 - `Dms\Common\Structure\DateTime\TimezonedDateTimeRange` - A date and time in a particular timezone range value `eg 2000-10-12 06:12:56 Australia/Melbourne until 2002-03-05 14:40:23 UTC`


### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Common\Structure\DateTime\DateRange;
use Dms\Common\Structure\DateTime\DateTimeRange;
use Dms\Common\Structure\DateTime\TimeRange;
use Dms\Common\Structure\DateTime\TimezonedDateTimeRange;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Holiday extends Entity
{
    const DATE_RANGE = 'dateRange';
    const TIME_RANGE = 'timeRange';
    const DATETIME_RANGE = 'dateTimeRange';
    const TIMEZONED_DATETIME_RANGE = 'timezoneDateTimeRange';

    /**
     * @var DateRange
     */
    public $dateRange;

    /**
     * @var TimeRange
     */
    public $timeRange;

    /**
     * @var DateTimeRange
     */
    public $dateTimeRange;

    /**
     * @var TimezonedDateTimeRange
     */
    public $timezoneDateTimeRange;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->dateRange)->asObject(DateRange::class);

        $class->property($this->timeRange)->asObject(TimeRange::class);

        $class->property($this->dateTimeRange)->asObject(DateTimeRange::class);

        $class->property($this->timezoneDateTimeRange)->asObject(TimezonedDateTimeRange::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types=1);

namespace App\Infrastructure\Persistence;

use App\Domain\Entities\Holiday;
use Dms\Common\Structure\DateTime\Persistence\DateRangeMapper;
use Dms\Common\Structure\DateTime\Persistence\DateTimeRangeMapper;
use Dms\Common\Structure\DateTime\Persistence\TimeRangeMapper;
use Dms\Common\Structure\DateTime\Persistence\TimezonedDateTimeRangeMapper;
use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;

/**
 * The App\Domain\Entities\Holiday entity mapper.
 */
class HolidayMapper extends EntityMapper
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
        $map->type(Holiday::class);
        $map->toTable('holidays');

        $map->idToPrimaryKey('id');

        $map->embedded(Holiday::DATE_RANGE)
            ->using(new DateRangeMapper('start_date', 'end_date'));

        $map->embedded(Holiday::TIME_RANGE)
            ->using(new TimeRangeMapper('time_time', 'end_time'));

        $map->embedded(Holiday::DATETIME_RANGE)
            ->using(new DateTimeRangeMapper('start_date_time', 'end_date_time'));

        $map->embedded(Holiday::TIMEZONED_DATETIME_RANGE)
            ->using(new TimezonedDateTimeRangeMapper('start_date_time', 'start_timezone', 'end_date_time', 'end_timezone'));
    }
}
```

### Module Configuration

```php
<?php declare(strict_types=1);

namespace App\Cms\Modules;

use App\Domain\Entities\Holiday;
use App\Domain\Services\Persistence\IHolidayRepository;
use Dms\Common\Structure\Field;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;

/**
 * The holiday module.
 */
class HolidayModule extends CrudModule
{
    public function __construct(IHolidayRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('holidays');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('date_range', 'Date Range')->dateRange()->required()
                )->bindToProperty(Holiday::DATE_RANGE),
                //
                $form->field(
                    Field::create('time_range', 'Time Range')->timeRange()->required()
                )->bindToProperty(Holiday::TIME_RANGE),
                //
                $form->field(
                    Field::create('date_time_range', 'Date Time Range')->dateTimeRange()->required()
                )->bindToProperty(Holiday::DATETIME_RANGE),
                //
                $form->field(
                    Field::create('timezoned_date_time_range', 'Timezoned Date Time Range')->dateTimeWithTimezoneRange()->required()
                )->bindToProperty(Holiday::TIMEZONED_DATETIME_RANGE),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Holiday::DATE_RANGE)->to(Field::create('date_range', 'Date Range')->dateRange()->required());
            $table->mapProperty(Holiday::TIME_RANGE)->to(Field::create('time_range', 'Time Range')->timeRange()->required());
            $table->mapProperty(Holiday::DATETIME_RANGE)->to(Field::create('date_time_range', 'Date Time Range')->dateTimeRange()->required());
            $table->mapProperty(Holiday::TIMEZONED_DATETIME_RANGE)->to(Field::create('timezoned_date_time_range', 'Timezone Date Time Range')->dateTimeWithTimezoneRange()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```