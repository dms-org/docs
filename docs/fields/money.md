Money
=====

Monetary values can be modelled using a value object:
 
 - `Dms\Common\Structure\Money\Money` - Represents an amount of currency `$123.45 AUD`
 
### Entity Structure

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Common\Structure\Money\Money;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Product extends Entity
{
    const PRICE = 'price';

    /**
     * @var Money
     */
    public $price;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->price)->asObject(Money::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Product;
use Dms\Common\Structure\Money\Persistence\MoneyMapper;

/**
 * The App\Domain\Entities\Product entity mapper.
 */
class ProductMapper extends EntityMapper
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
        $map->type(Product::class);
        $map->toTable('products');

        $map->idToPrimaryKey('id');

        $map->embedded(Product::PRICE)
            ->using(new MoneyMapper('price_amount', 'price_currency'));
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
use App\Domain\Services\Persistence\IProductRepository;
use App\Domain\Entities\Product;
use Dms\Common\Structure\Field;

/**
 * The product module.
 */
class ProductModule extends CrudModule
{
    public function __construct(IProductRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('product');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('price', 'Price')->money()->required()
                )->bindToProperty(Product::PRICE),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Product::PRICE)->to(Field::create('price', 'Price')->money()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```