Files
=====

Files and images can be modelled using value objects.

 - `Dms\Common\Structure\FileSystem\File`
 - `Dms\Common\Structure\FileSystem\Image`

These classes wrap an absolute file path which can references a file stored on disk or externally.

### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Common\Structure\FileSystem\File;
use Dms\Common\Structure\FileSystem\Image;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class House extends Entity
{
    const FLOORPLAN = 'floorplan';
    const PHOTO = 'photo';

    /**
     * @var File
     */
    public $floorplan;

    /**
     * @var Image
     */
    public $photo;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->floorplan)->asObject(File::class);

        $class->property($this->photo)->asObject(Image::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\House;
use Dms\Common\Structure\FileSystem\Persistence\FileMapper;
use Dms\Common\Structure\FileSystem\Persistence\ImageMapper;

/**
 * The App\Domain\Entities\House entity mapper.
 */
class HouseMapper extends EntityMapper
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
        $map->type(House::class);
        $map->toTable('houses');

        $map->idToPrimaryKey('id');

        $map->embedded(House::FLOORPLAN)
            ->using(new FileMapper('floorplan', 'floorplan_file_name', public_path('app/house-files')));

        $map->embedded(House::PHOTO)
            ->using(new ImageMapper('photo', 'photo_file_name', public_path('app/house-files')));
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
use App\Domain\Services\Persistence\IHouseRepository;
use App\Domain\Entities\House;
use Dms\Common\Structure\Field;

/**
 * The house module.
 */
class HouseModule extends CrudModule
{
    public function __construct(IHouseRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('house');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('floorplan', 'Floorplan')
                        ->file()
                        ->required()
                        ->moveToPathWithRandomFileName(public_path('app/house-files'))
                )->bindToProperty(House::FLOORPLAN),
                //
                $form->field(
                    Field::create('photo', 'Photo')
                        ->image()
                        ->required()
                        ->moveToPathWithRandomFileName(public_path('app/house-files'))
                )->bindToProperty(House::PHOTO),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```