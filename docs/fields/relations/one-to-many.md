One to Many
===========

An one to many bi-directional relationship between entities can be defined as follows.

### Entity Structure

#### app/Domain/Entities/Mother.php

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\EntityCollection;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Mother extends Entity
{
    const NAME = 'name';
    const CHILDREN = 'children';

    /**
     * @var string
     */
    public $name;

    /**
     * @var EntityCollection|Child[]
     */
    public $children;

    /**
     * Parent constructor.
     */
    public function __construct()
    {
        parent::__construct();
        $this->children = Child::collection();
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->children)->asType(Child::collectionType());
    }
}
```

#### app/Domain/Entities/Child.php

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Child extends Entity
{
    const NAME = 'name';
    const MOTHER = 'mother';

    /**
     * @var string
     */
    public $name;

    /**
     * @var Mother
     */
    public $mother;

    /**
     * Child constructor.
     *
     * @param string $name
     * @param Mother $mother
     */
    public function __construct($name, Mother $mother)
    {
        parent::__construct();
        $this->name   = $name;
        $this->mother = $mother;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->mother)->asObject(Mother::class);
    }
}
```


### Mapper Configuration

#### app/Infrastructure/Persistence/MotherMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Mother;
use App\Domain\Entities\Child;

/**
 * The App\Domain\Entities\Mother entity mapper.
 */
class MotherMapper extends EntityMapper
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
        $map->type(Mother::class);
        $map->toTable('mothers');

        $map->idToPrimaryKey('id');

        $map->property(Mother::NAME)->to('name')->asVarchar(255);

        $map->relation(Mother::CHILDREN)
            ->to(Child::class)
            ->toMany()
            ->identifying()
            ->withBidirectionalRelation(Child::MOTHER)
            ->withParentIdAs('mother_id');
    }
}
```

#### app/Infrastructure/Persistence/ChildMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Child;
use App\Domain\Entities\Mother;

/**
 * The App\Domain\Entities\Child entity mapper.
 */
class ChildMapper extends EntityMapper
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
        $map->type(Child::class);
        $map->toTable('children');

        $map->idToPrimaryKey('id');

        $map->property(Child::NAME)->to('name')->asVarchar(255);

        $map->column('mother_id')->asUnsignedInt();
        $map->relation(Child::MOTHER)
            ->to(Mother::class)
            ->manyToOne()
            ->withBidirectionalRelation(Mother::CHILDREN)
            ->withRelatedIdAs('mother_id');
    }
}
```

### Module Configuration

#### app/Cms/Modules/MotherModule.php

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IMotherRepository;
use App\Domain\Entities\Mother;
use Dms\Common\Structure\Field;
use App\Domain\Services\Persistence\IChildRepository;
use App\Domain\Entities\Child;

/**
 * The mother module.
 */
class MotherModule extends CrudModule
{
    /**
     * @var IChildRepository
     */
    protected $childRepository;

    public function __construct(IMotherRepository $dataSource, IAuthSystem $authSystem, IChildRepository $childRepository)
    {
        $this->childRepository = $childRepository;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('mother');

        $module->labelObjects()->fromProperty(Mother::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(Mother::NAME),
                //
                $form->field(
                    Field::create('children', 'Children')
                        ->entitiesFrom($this->childRepository)
                        ->labelledBy(Child::NAME)
                        ->mapToCollection(Child::collectionType())
                        // Add this line if you want to load the
                        // options asynchronously via autocomplete
                        ->searchableBy(Child::NAME)
                )->bindToProperty(Mother::CHILDREN),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Mother::NAME)->to(Field::create('name', 'Name')->string()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```

#### app/Cms/Modules/ChildModule.php

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IChildRepository;
use App\Domain\Entities\Child;
use Dms\Common\Structure\Field;
use App\Domain\Services\Persistence\IMotherRepository;
use App\Domain\Entities\Mother;

/**
 * The child module.
 */
class ChildModule extends CrudModule
{
    /**
     * @var IMotherRepository
     */
    protected $motherRepository;


    public function __construct(IChildRepository $dataSource, IAuthSystem $authSystem, IMotherRepository $motherRepository)
    {
        $this->motherRepository = $motherRepository;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('child');

        $module->labelObjects()->fromProperty(Child::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(Child::NAME),
                //
                $form->field(
                    Field::create('mother', 'Mother')
                        ->entityFrom($this->motherRepository)
                        ->required()
                        ->labelledBy(Mother::NAME)
                )->bindToProperty(Child::MOTHER),
            ]);
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Child::NAME)->to(Field::create('name', 'Name')->string()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```