Custom Fields
=============

If you need a field in your backend that is not provided out of the box you can implement
a custom field renderer in your app which will override how a field is displayed.

### app/Cms/Modules/YourModule.php

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Entities\Entity;
use Dms\Common\Structure\Field;

/**
 * The photograph module.
 */
class YourModule extends CrudModule
{

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('your-module');

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('custom_field', 'Custom Field')->string()->required()
                )->bindToProperty(Entity::PROPERTY),
                //
            ]);
        });

    }
}
```

### app/Cms/Custom/CustomFieldRenderer.php

```php
<?php declare(strict_types=1);

namespace App\Cms\Custom;

use Dms\Core\Form\Field\Type\ArrayOfType;
use Dms\Core\Form\Field\Type\StringType;
use Dms\Core\Form\IField;
use Dms\Core\Form\IFieldType;
use Dms\Web\Laravel\Renderer\Form\Field\BladeFieldRenderer;
use Dms\Web\Laravel\Renderer\Form\FormRenderingContext;


/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class CustomFieldRenderer extends BladeFieldRenderer
{

    /**
     * Gets the expected class of the field type for the field.
     *
     * @return array
     */
    public function getFieldTypeClasses(): array
    {
        return [StringType::class];
    }

    /**
     * @param FormRenderingContext $renderingContext
     * @param IField               $field
     * @param IFieldType           $fieldType
     *
     * @return bool
     */
    protected function canRender(FormRenderingContext $renderingContext, IField $field, IFieldType $fieldType): bool
    {
        return $renderingContext->getAction()->getPackageName() === 'your-package'
            && $renderingContext->getAction()->getModuleName() === 'your-module'
            && $field->getName() === 'custom_field';
    }

    /**
     * @param FormRenderingContext $renderingContext
     * @param IField               $field
     * @param IFieldType           $fieldType
     *
     * @return string
     */
    protected function renderField(FormRenderingContext $renderingContext, IField $field, IFieldType $fieldType): string
    {
        return view('dms.custom-fields.text', [
            'field' => $field,
        ])->render();
    }

    /**
     * @param FormRenderingContext $renderingContext
     * @param IField               $field
     * @param mixed                $value
     * @param IFieldType           $fieldType
     *
     * @return string
     */
    protected function renderFieldValue(FormRenderingContext $renderingContext, IField $field, $value, IFieldType $fieldType): string
    {
        return view('dms.custom-fields.readonly-text', [
            'field' => $field,
            'value' => $value,
        ])->render();
    }
}
```

### config/dms.php

Register your custom form field renderer in `config/dms.php`

```php
<?php
[
    // ...
    'services' => [
        // ...
        'renderers' => [
            // ...
            'form-fields' => [
                // Add your class here
                App\Cms\Custom\CustomFieldRenderer::class,
                // ...
            ],
            // ...
        ],
    ],
]
```

### resources/views/dms/custom-fields/text.blade.php

```blade
Your custom field:
<input type="text" name="{{ $field->getName() }}" value="{{ $field->getInitialValue() }}" />
```

### resources/views/dms/custom-fields/readonly-text.blade.php

```blade
Your custom value: {{ $value }}
```