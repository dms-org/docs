Dependent Fields
================

Often static form builders are quite limiting and dont provide much of the functionality required for even
a semi-complex CMS. 

Dependent fields offer a powerful way to build forms that contain complex conditionals, multiple steps or
can even be (ab)used to provide live calculations or previews.

### Multiple steps example

Here is an example of a form which will show different options for the second field based on the value of the first field.

```php
<?php declare(strict_types=1);

namespace App\Cms\Modules;

use Dms\Common\Structure\Field;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;

class YourModule extends CrudModule
{
    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        // ...

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('type', 'Type')->string()->required()->oneOf([
                        'fruit'     => 'Fruit',
                        'vegetable' => 'Vegetable',
                    ])
                )->withoutBinding(),
            ]);

            $form->dependentOn(['type'], function (CrudFormDefinition $form, array $input) {

                if ($input['type'] === 'fruit') {
                    $form->continueSection([
                        $form->field(
                            Field::create('type_of_fruit', 'Type Of Fruit')->string()->required()->oneOf([
                                'apple'  => 'Apple',
                                'orange' => 'Orange',
                            ])
                        )->withoutBinding(),
                    ]);
                }

                if ($input['type'] === 'vegetable') {
                    $form->continueSection([
                        $form->field(
                            Field::create('type_of_vegetable', 'Type Of Vegetable')->string()->required()->oneOf([
                                'carrot' => 'Carrot',
                                'potato' => 'Potato',
                            ])
                        )->withoutBinding(),
                    ]);
                }

            });
        });

        // ...
    }
}
```

### Live calculation example

Here is an example of a form which will sum two integers and display the result.

```php
<?php declare(strict_types=1);

namespace App\Cms\Modules;

use Dms\Common\Structure\Field;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;

class YourModule extends CrudModule
{
    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        // ...

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('a', 'A')->int()->required()
                )->withoutBinding(),
                //
                $form->field(
                    Field::create('b', 'B')->int()->required()
                )->withoutBinding(),
            ]);

            $form->dependentOn(['a', 'b'], function (CrudFormDefinition $form, array $input) {

                $form->continueSection([
                    $form->field(
                        Field::create('result', 'Result')->int()->value($input['a'] + $input['b'])->readonly()
                    )->withoutBinding(),
                ]);

            });
        });

        // ...
    }
}
```