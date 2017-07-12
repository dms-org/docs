Arrays/Collections
==================

It is often useful to represent an array of values. 

For scalar values you can use a native `array`.

For a collection of value objects it is recommended you use `Dms\Core\Model\ValueObjectCollection`.

For a collection of entities [see relations](./relations/index.md).

### Entity Structure

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Common\Structure\Web\EmailAddress;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;
use Dms\Core\Model\Type\Builder\Type;
use Dms\Core\Model\ValueObjectCollection;

class Contact extends Entity
{
    const NAMES = 'NAMES';
    const EMAIL_ADDRESSES = 'emailAddresses';

    /**
     * @var string[]
     */
    public $names;

    /**
     * @var ValueObjectCollection|EmailAddress[]
     */
    public $emailAddresses;


    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->names)->asArrayOf(Type::string());

        $class->property($this->emailAddresses)->asType(EmailAddress::collectionType());
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Contact;
use Dms\Common\Structure\Web\Persistence\EmailAddressMapper;

/**
 * The App\Domain\Entities\Contact entity mapper.
 */
class ContactMapper extends EntityMapper
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
        $map->type(Contact::class);
        $map->toTable('contacts');

        $map->idToPrimaryKey('id');

        // Map names array to JSON string in 'names' column
        $map->property(Contact::NAMES)
            ->mappedVia(
                function (array $names) {
                    return json_encode($names);
                },
                function (string $json) {
                    return (array)json_decode($json);
                }
            )
            ->to('names')
            ->asVarchar(8000);

        // Map email addresses to separate table 'contact_email_addresses'
        $map->embeddedCollection(Contact::EMAIL_ADDRESSES)
            ->toTable('contact_email_addresses')
            ->withPrimaryKey('id')
            ->withForeignKeyToParentAs('contact_id')
            ->using(new EmailAddressMapper('email_addresses'));
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
use App\Domain\Services\Persistence\IContactRepository;
use App\Domain\Entities\Contact;
use Dms\Common\Structure\Field;

/**
 * The contact module.
 */
class ContactModule extends CrudModule
{
    public function __construct(IContactRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('contact');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('names', 'Names')->arrayOf(
                        Field::element()->string()->required()
                    )
                )->bindToProperty('names'),
                //
                $form->field(
                    Field::create('email_addresses', 'Email Addresses')->arrayOf(
                        Field::element()->email()->required()
                    )
                )->bindToProperty(Contact::EMAIL_ADDRESSES),
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