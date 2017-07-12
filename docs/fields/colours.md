Colours
=======

Colours can be modelled using value objects:

 - `Dms\Common\Structure\Colour\Colour` - A RGB colour value `rgb(123, 123, 123)`
 - `Dms\Common\Structure\Colour\TransparentColour` - A RGBA colour value `rgba(123, 123, 123, 0.5)`
 
### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Common\Structure\Colour\Colour;
use Dms\Common\Structure\Colour\TransparentColour;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Photograph extends Entity
{
    const BACKGROUND = 'background';
    const FOREGROUND = 'foreground';

    /**
     * @var Colour
     */
    public $background;

    /**
     * @var TransparentColour
     */
    public $foreground;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->background)->asObject(Colour::class);

        $class->property($this->foreground)->asObject(TransparentColour::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Photograph;
use Dms\Common\Structure\Colour\Mapper\ColourMapper;
use Dms\Common\Structure\Colour\Mapper\TransparentColourMapper;

/**
 * The App\Domain\Entities\Photograph entity mapper.
 */
class PhotographMapper extends EntityMapper
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
        $map->type(Photograph::class);
        $map->toTable('photographs');

        $map->idToPrimaryKey('id');

        $map->embedded(Photograph::BACKGROUND)
            ->using(ColourMapper::asHexString('background'));

        $map->embedded(Photograph::FOREGROUND)
            ->using(TransparentColourMapper::asRgbaString('foreground'));
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
use App\Domain\Services\Persistence\IPhotographRepository;
use App\Domain\Entities\Photograph;
use Dms\Common\Structure\Field;

/**
 * The photograph module.
 */
class PhotographModule extends CrudModule
{
    public function __construct(IPhotographRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('photograph');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('background', 'Background')->colour()->required()
                )->bindToProperty(Photograph::BACKGROUND),
                //
                $form->field(
                    Field::create('foreground', 'Foreground')->colourWithTransparency()->required()
                )->bindToProperty(Photograph::FOREGROUND),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Photograph::BACKGROUND)->to(Field::create('background', 'Background')->colour()->required());
            $table->mapProperty(Photograph::FOREGROUND)->to(Field::create('foreground', 'Foreground')->colourWithTransparency()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```