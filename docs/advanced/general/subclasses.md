Subclasses
==========

A powerful of feature of OOP to help remove duplication is subclassing.
Subclassing allowing you to create inheritance trees to better model your domain.

### Entity Structure

We'll model this example from the following diagram:

![Class Hierarchy](/resources/images/architecture/example-class-hierarchy-1.png)

#### app/Domain/Entities/Player.php

```php
<?php

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

abstract class Player extends Entity
{
    const NAME = 'name';
    
    /**
     * @var string
     */
    public $name;

    public function __construct(string $name)
    {
        parent::__construct();
        $this->name = $name;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();
    }
}
```

#### app/Domain/Entities/Footballer.php

```php
<?php

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;

class Footballer extends Player
{
    const CLUB = 'club';
    
    /**
     * @var string
     */
    public $club;

    public function __construct(string $name, string $club)
    {
        parent::__construct($name);
        $this->club = $club;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        parent::defineEntity($class);

        $class->property($this->club)->asString();
    }
}
```

#### app/Domain/Entities/Cricketer.php

```php
<?php

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;

class Cricketer extends Player
{
    const BATTING_AVERAGE = 'battingAverage';
    
    /**
     * @var int
     */
    public $battingAverage;

    public function __construct(string $name, int $battingAverage)
    {
        parent::__construct($name);
        $this->battingAverage = $battingAverage;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        parent::defineEntity($class);

        $class->property($this->battingAverage)->asInt();
    }
}
```

#### app/Domain/Entities/Bowler.php

```php
<?php

namespace App\Domain\Entities;

use Dms\Core\Model\Object\ClassDefinition;

class Bowler extends Cricketer
{
    const BOWLING_AVERAIGE = 'bowlingAverage';
    
    /**
     * @var int
     */
    public $bowlingAverage;

    public function __construct(string $name, int $battingAverage, int $bowlingAverage)
    {
        parent::__construct($name, $battingAverage);
        $this->bowlingAverage = $bowlingAverage;
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        parent::defineEntity($class);

        $class->property($this->bowlingAverage)->asInt();
    }
}
```

### Module Configuration

Modules support mapping to entity subclasses as shown in the following example

```php
<?php declare(strict_types=1);

namespace App\Cms\Modules;

use App\Domain\Entities\Bowler;
use App\Domain\Entities\Cricketer;
use App\Domain\Entities\Footballer;
use App\Domain\Entities\Player;
use Dms\Common\Structure\Field;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;

/**
 * The player module.
 */
class PlayerModule extends CrudModule
{
    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('player');

        $module->labelObjects()->fromProperty(Player::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $typeField = Field::create('type', 'Type')->string()->oneOf([
                'footballer' => 'Footballer',
                'cricketer'  => 'Cricketer',
                'bowler'     => 'Bowler',
            ]);

            if (!$form->isCreateForm()) {
                // Cannot change the type of the entity
                $typeField->readonly();
            }

            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(Player::NAME),
                //
                $form->field(
                    $typeField
                )->bindToCallbacks(function (Player $player) {
                    return [
                           Footballer::class => 'footballer',
                           Cricketer::class  => 'cricketer',
                           Bowler::class     => 'bowler',
                       ][get_class($player)];
                }, function () {
                    // Unused
                }),
            ]);

            $form->dependentOn(['type'], function (CrudFormDefinition $form, array $input) {
                if ($input['type'] === 'footballer') {
                    $form->mapToSubClass(Footballer::class);

                    $form->continueSection([
                        $form->field(
                            Field::create('club', 'Club')->string()->required()
                        )->bindToProperty(Footballer::CLUB)
                    ]);
                }

                if ($input['type'] === 'cricketer' || $input['type'] === 'bowler') {
                    if ($input['type'] === 'cricketer') {
                        $form->mapToSubClass(Cricketer::class);
                    }

                    $form->continueSection([
                        $form->field(
                            Field::create('batting_average', 'Batting Average')->int()->required()
                        )->bindToProperty(Cricketer::BATTING_AVERAGE)
                    ]);

                    if ($input['type'] === 'bowler') {
                        $form->mapToSubClass(Bowler::class);

                        $form->continueSection([
                            $form->field(
                                Field::create('bowling_average', 'Bowling Average')->int()->required()
                            )->bindToProperty(Bowler::BOWLING_AVERAGE)
                        ]);
                    }
                }
            });
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Player::NAME)->to(Field::create('name', 'Name')->string());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```

## Mapper Configuration (Single Table Inheritance)

You can map all the subclasses to one database table using single table inheritance pattern with a type column.

```php
<?php declare(strict_types=1);

namespace App\Infrastructure\Persistence;

use App\Domain\Entities\Bowler;
use App\Domain\Entities\Cricketer;
use App\Domain\Entities\Footballer;
use App\Domain\Entities\Player;
use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;

/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class PlayerMapper extends EntityMapper
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
        $map->type(Player::class);
        $map->toTable('players');

        $map->idToPrimaryKey('id');

        $map->column('type')->asEnum(['footballer', 'cricketer', 'bowler']);
        $map->property(Player::NAME)->to('name')->asVarchar(255);

        $map->subclass()->withTypeInColumn('type', 'footballer')->define(function (MapperDefinition $map) {
            $map->type(Footballer::class);
            $map->property(Footballer::CLUB)->to('club')->asVarchar(255);
        });

        $map->subclass()->withTypeInColumn('type', 'cricketer')->define(function (MapperDefinition $map) {
            $map->type(Cricketer::class);
            $map->property(Cricketer::BATTING_AVERAGE)->to('batting_average')->asInt();

            $map->subclass()->withTypeInColumn('type', 'bowler')->define(function (MapperDefinition $map) {
                $map->type(Bowler::class);
                $map->property(Bowler::BOWLING_AVERAGE)->to('bowling_average')->asInt();
            });
        });
    }
}
```

## Mapper Configuration (Class Table Inheritance)

Alternatively, you can map each the subclasses to a separate table.

```php
<?php declare(strict_types=1);

namespace App\Infrastructure\Persistence;

use App\Domain\Entities\Bowler;
use App\Domain\Entities\Cricketer;
use App\Domain\Entities\Footballer;
use App\Domain\Entities\Player;
use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;

/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class PlayerMapper extends EntityMapper
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
        $map->type(Player::class);
        $map->toTable('players');

        $map->idToPrimaryKey('id');
        $map->property(Player::NAME)->to('name')->asVarchar(255);

        $map->subclass()->asSeparateTable('footballers')->define(function (MapperDefinition $map) {
            $map->type(Footballer::class);
            $map->property(Footballer::CLUB)->to('club')->asVarchar(255);
        });

        $map->subclass()->asSeparateTable('cricketers')->define(function (MapperDefinition $map) {
            $map->type(Cricketer::class);
            $map->property('battingAverage')->to('batting_average')->asInt();

            $map->subclass()->asSeparateTable('bowlers')->define(function (MapperDefinition $map) {
                $map->type(Bowler::class);
                $map->property(Bowler::BOWLING_AVERAGE)->to('bowling_average')->asInt();
            });
        });
    }
}
```