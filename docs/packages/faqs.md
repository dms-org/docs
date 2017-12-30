FAQ's Package
=============

It is often necessary to include a question and answer section in a website.
The FAQ package provides the functionality for this.

## Installation

### Install the package via composer

`composer require dms-org/package.faqs`

### Register the package in `app/AppCms.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Cms;
use Dms\Core\CmsDefinition;
use Dms\Package\Faq\Cms\FaqPackage;

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
            // Add this line to register the package...
            'faqs' => FaqPackage::class,
        ]);
    }
}
```

### Register the ORM in `app/AppOrm.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Persistence\Db\Mapping\Definition\Orm\OrmDefinition;
use Dms\Core\Persistence\Db\Mapping\Orm;
use Dms\Package\Faq\Persistence\FaqOrm;

/**
 * The application's orm.
 *
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class AppOrm extends Orm
{
    /**
     * Defines the object mappers registered in the orm.
     *
     * @param OrmDefinition $orm
     *
     * @return void
     */
    protected function define(OrmDefinition $orm)
    {
        // Add this line to register the mappers
        $orm->encompass(new FaqOrm());
    }
}
```

## Usage

### Load the FAQ's in your controller

```php
<?php declare(strict_types = 1);

namespace App\Http\Controllers;

use Dms\Package\Faq\Core\FaqLoaderService;

/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class FaqsController extends Controller
{
    public function showFaqsPage(FaqLoaderService $faqLoaderService)
    {
        return view('faqs', [
            'faqs' => $faqLoaderService->loadFaqs(),
        ]);
    }
}
```

### Display the FAQ's in your view

```html
<article>
    <header>
        <h1>FAQ's</h1>
    </header>
    <section class="content">
        @foreach($faqs as $faq)
            <div>
                <strong>{{ $faq->question }}</strong>
                <div>{!! $faq->answer->asString() !!}</div>
            </div>
        @endforeach
    </section>
</article>
```
