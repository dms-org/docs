Web
===

Miscellaneous web-related fields can be modelled using the value objects:

 - `Dms\Common\Structure\Web\EmailAddress` - An email address `john@gmail.com`
 - `Dms\Common\Structure\Web\IpAddress` - An IP address `123.123.123.123`
 - `Dms\Common\Structure\Web\Url` - An absolute url `https://www.google.com`
 - `Dms\Common\Structure\Web\Html` - A raw HTML string `<p>abc</p>`

### Entity Structure

```php
<?php declare(strict_types = 1);

namespace App\Domain\Entities;

use Dms\Common\Structure\Web\EmailAddress;
use Dms\Common\Structure\Web\Html;
use Dms\Common\Structure\Web\IpAddress;
use Dms\Common\Structure\Web\Url;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Business extends Entity
{
    const EMAIL_ADDRESS = 'emailAddress';
    const IP_ADDRESS = 'ipAddress';
    const WEBSITE = 'website';
    const DESCRIPTION = 'description';

    /**
     * @var EmailAddress
     */
    public $emailAddress;

    /**
     * @var IpAddress
     */
    public $ipAddress;

    /**
     * @var Url
     */
    public $website;

    /**
     * @var Html
     */
    public $description;

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->emailAddress)->asObject(EmailAddress::class);

        $class->property($this->ipAddress)->asObject(IpAddress::class);

        $class->property($this->website)->asObject(Url::class);

        $class->property($this->description)->asObject(Html::class);
    }
}
```

### Mapper Configuration

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Business;
use Dms\Common\Structure\Web\Persistence\EmailAddressMapper;
use Dms\Common\Structure\Web\Persistence\IpAddressMapper;
use Dms\Common\Structure\Web\Persistence\UrlMapper;
use Dms\Common\Structure\Web\Persistence\HtmlMapper;

/**
 * The App\Domain\Entities\Business entity mapper.
 */
class BusinessMapper extends EntityMapper
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
        $map->type(Business::class);
        $map->toTable('businesses');

        $map->idToPrimaryKey('id');

        $map->embedded(Business::EMAIL_ADDRESS)
            ->using(new EmailAddressMapper('email_address'));

        $map->embedded(Business::IP_ADDRESS)
            ->using(new IpAddressMapper('ip_address'));

        $map->embedded(Business::WEBSITE)
            ->using(new UrlMapper('website'));

        $map->embedded(Business::DESCRIPTION)
            ->using(new HtmlMapper('description'));
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
use App\Domain\Services\Persistence\IBusinessRepository;
use App\Domain\Entities\Business;
use Dms\Common\Structure\Field;

/**
 * The business module.
 */
class BusinessModule extends CrudModule
{
    public function __construct(IBusinessRepository $dataSource, IAuthSystem $authSystem)
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
        $module->name('business');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('email_address', 'Email Address')->email()->required()
                )->bindToProperty(Business::EMAIL_ADDRESS),
                //
                $form->field(
                    Field::create('ip_address', 'Ip Address')->ipAddress()->required()
                )->bindToProperty(Business::IP_ADDRESS),
                //
                $form->field(
                    Field::create('website', 'Website')->url()->required()
                )->bindToProperty(Business::WEBSITE),
                //
                $form->field(
                    Field::create('description', 'Description')->html()->required()
                )->bindToProperty(Business::DESCRIPTION),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Business::EMAIL_ADDRESS)->to(Field::create('email_address', 'Email Address')->email()->required());
            $table->mapProperty(Business::IP_ADDRESS)->to(Field::create('ip_address', 'Ip Address')->ipAddress()->required());
            $table->mapProperty(Business::WEBSITE)->to(Field::create('website', 'Website')->url()->required());
            $table->mapProperty(Business::DESCRIPTION)->to(Field::create('description', 'Description')->html()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```