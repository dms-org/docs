Extending Modules
=================

It may become necessary to extend the functionality of a module which is installed as a standalone package.
To achieve this, you can hook into to the events emitted and add the desired functionality.

You can add these event listeners directly in your `app/AppCms.php` or extract them into their
own classes if applicable.

## Adding form fields

Here is an example of adding an additional colour field to the products module from the shop package.

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Common\Structure\Field;
use Dms\Core\Cms;
use Dms\Core\CmsDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Package\Analytics\AnalyticsPackage;
use Dms\Package\Shop\Cms\ShopPackage;
use Dms\Package\Shop\Domain\Entities\Product\Product;
use Dms\Web\Laravel\Auth\AdminPackage;
use Dms\Web\Laravel\Document\PublicFilePackage;

/**
 * The application's cms.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppCms extends Cms
{
    /**
     * Defines the structure and installed packages of the cms.
     *
     * @param CmsDefinition $cms
     *
     * @return void
     */
    protected function define(CmsDefinition $cms)
    {
        $cms->packages([
            'admin'      => AdminPackage::class,
            'documents'  => PublicFilePackage::class,
            'analytics'  => AnalyticsPackage::class,
            'shop'       => ShopPackage::class,
        ]);

        \Event::listen('dms::shop.products.defined-form', function (CrudFormDefinition $form) {
            
            // Add your form fields here
            $form->continueSection([
                $form->field(
                    Field::create('colour', 'Colour')->string()->required()
                )->bindToCallbacks(
                    function (Product $product) {
                        return $product->metadata['colour'] ?? null;
                    },
                    function (Product $product, string $value) {
                        $product->metadata['colour'] = $value;
                    }
                ),
            ]);
            
        });

    }
}
```

## Adding actions

Here is an example of adding a custom action to the product module.

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Common\Structure\Field;
use Dms\Core\Cms;
use Dms\Core\CmsDefinition;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Package\Analytics\AnalyticsPackage;
use Dms\Package\Shop\Cms\ShopPackage;
use Dms\Web\Laravel\Auth\AdminPackage;
use Dms\Web\Laravel\Document\PublicFilePackage;

/**
 * The application's cms.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppCms extends Cms
{
    /**
     * Defines the structure and installed packages of the cms.
     *
     * @param CmsDefinition $cms
     *
     * @return void
     */
    protected function define(CmsDefinition $cms)
    {
        $cms->packages([
            'admin'      => AdminPackage::class,
            'documents'  => PublicFilePackage::class,
            'analytics'  => AnalyticsPackage::class,
            'shop'       => ShopPackage::class,
        ]);

        \Event::listen('dms::shop.products.define', function (CrudModuleDefinition $module) {
            // Add your custom action here
            $module->action('custom-action')
                ->handler(function (array $input) {
                    // ...
                });
        });

    }
}
```