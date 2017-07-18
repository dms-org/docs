Content Package
===============

Sometimes it is unnecessary to define fully-blown entities and mappers for structures in your app.
The content package provides a generic structure that you can use to define the schema of the content of your app.

## Installation

### Install the package via composer

`composer require dms-org/package.content`

### Register the ORM in `app/AppORM.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Persistence\Db\Mapping\Definition\Orm\OrmDefinition;
use Dms\Core\Persistence\Db\Mapping\Orm;
use Dms\Package\Content\Persistence\ContentOrm;

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
        // Add this line to register the content mappers
        $orm->encompass(new ContentOrm($this->iocContainer));
    }
}
```

### Register the view composer

Register a view composer in the `boot` method of `app/Providers/AppServiceProvider.php` to resolve
`Dms\Package\Content\Core\ContentLoaderService`.

```php
<?php

namespace App\Providers;

use Dms\Package\Content\Core\ContentLoaderService;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        // Ensure the content can be loaded in all views
        // via $contentLoader
        \view()->composer('*', function ($view) {
            $view->with('contentLoader', app(ContentLoaderService::class));
        });
    }
    
    // Omitted...
}

```

## Usage

### Defining your content schema

Create a custom content package in `app/Cms/MyContentPackage.php`

```php
<?php declare(strict_types = 1);

namespace App\Cms;

use App\Http\Controllers\PageController;
use Dms\Core\ICms;
use Dms\Package\Content\Cms\ContentPackage;
use Dms\Package\Content\Cms\Definition\ContentConfigDefinition;
use Dms\Package\Content\Cms\Definition\ContentGroupDefiner;
use Dms\Package\Content\Cms\Definition\ContentModuleDefinition;
use Dms\Package\Content\Cms\Definition\ContentPackageDefinition;

/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class MyContentPackage extends ContentPackage
{
    protected static function defineConfig(ContentConfigDefinition $config)
    {
        $config
            ->storeImagesUnder(public_path('app/content/images'))
            ->mappedToUrl(url('app/content/images'))
            ->storeFilesUnder(public_path('app/content/files'));
    }

    /**
     * Defines the structure of the content.
     *
     * @param ContentPackageDefinition $content
     *
     * @return void
     */
    protected function defineContent(ContentPackageDefinition $content)
    {
        $content->module('pages', 'file-text', function (ContentModuleDefinition $module) {

            $module->group('home', 'Home')
                ->withText('title', 'Title')
                ->withHtml('content', 'Content')
                ->withArrayOf('carousel-item', 'Hero Carousel', function (ContentGroupDefiner $item) {
                    $item
                        ->withText('caption', 'Caption')
                        ->withImage('image', 'Image');
                })
                ->withMetadata('title', 'Meta - Title')
                ->withMetadata('description', 'Meta - Description')
                // Optionally define a preview callback to enable
                // live content previews in the CMS
                ->setPreviewCallback(function () {
                    return \app()->call(PageController::class . '@showHomePage')->render();
                });
            
            // Add more pages here ...
        });

        // Define additional modules here...

        // Optionally, register your own custom modules if you want to include
        // them in your content package...
        $content->customModules([
            'custom' => CustomModule::class
        ]);
    }
}
```

### Register the package in `app/AppCms.php`

```php
<?php declare(strict_types = 1);

namespace App;

use App\Cms\MyContentPackage;
use Dms\Core\Cms;
use Dms\Core\CmsDefinition;

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
            // Add this line to register your content package...
            'content' => MyContentPackage::class,
        ]);
    }
}
```


### Load the content in your views

You can then use the `$contentLoader` in your blade files to retrieve the content from the database.

```html
<?php $content = $contentLoader->load('pages.home') ?>
<html>
<head>
    <title>{{ $content->getMetadata('title') }}</title>
    <meta name="description" content="{{ $content->getMetadata('title') }}">
</head>
<body>
<header>
    <h1>{{ $content->getText('title') }}</h1>
</header>
<article>
    {!! $content->getHtml('content') !!}
</article>
<ul class="carousel">
    @foreach($content->getArrayOf('carousel-item') as $item)
        <li>
            <img src="{{ $item->hasImage('image') ? asset_file_url($item->getImage('image')) : 'http://placehold.it/500x500' }}" />
            <span>{{ $item->getText('caption') }}</span>
        </li>
    @endforeach
</ul>
</body>
</html>
```