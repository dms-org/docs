Blog Package
============

The blog package provides the functionality for building a simple blogging platform.

## Installation

### Install the package via composer

`composer require dms/package.blog`

### Register the package in `app/AppCms.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Cms;
use Dms\Core\CmsDefinition;
use Dms\Package\Blog\Cms\BlogPackage;

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
            // Add this line to register the blog package...
            'blog' => BlogPackage::class,
        ]);
    }
}
```

### Register the ORM in `app/AppORM.php`

```php
<?php declare(strict_types = 1);

namespace App;

use Dms\Core\Persistence\Db\Mapping\Definition\Orm\OrmDefinition;
use Dms\Core\Persistence\Db\Mapping\Orm;
use Dms\Package\Blog\Infrastructure\Persistence\BlogOrm;

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
        // Add this line to register the blog mappers
        $orm->encompass((new BlogOrm($this->iocContainer))->inNamespace('blog_'));
    }
}
```

### Configure the blog package in `app/Providers/AppServiceProvider.php`

```php
<?php

namespace App\Providers;

use App\AppCms;
use App\AppOrm;
use Dms\Core\ICms;
use Dms\Core\Persistence\Db\Mapping\IOrm;
use Dms\Package\Blog\Domain\Entities\BlogArticle;
use Dms\Package\Blog\Domain\Services\Config\BlogConfiguration;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        // ...

        // Register your blog configuration here...
        $this->app->bind(BlogConfiguration::class, function () {
            return BlogConfiguration::builder()
                ->setFeaturedImagePath(public_path('app/images/blog'))
                ->useDashedSlugGenerator()
                // Supply a preview callback to provide article previews
                // directly from the backend. This can be omitted to disable this feature.
                ->setArticlePreviewCallback(function (BlogArticle $article) {
                    return view('blog.article', ['article' => $article])->render();
                })
                ->build();
        });
    }
}
```

## Usage

Here is an example of how to utilise the default functionality of the blog package.

```php
<?php declare(strict_types = 1);

use Dms\Common\Structure\Web\EmailAddress;
use Dms\Package\Blog\Domain\Services\BlogKernel;

function demonstrateBlogFunctionality(BlogKernel $blog)
{
    $categories = $blog->categories()->getAll();

    $specificCategory = $blog->categories()->loadFromSlug('news');

    $articles = $blog->articles()->getAll();

    $articles = $blog->articles()->getPage(1, 15);

    $specificArticle = $blog->articles()->loadFromSlug('a-new-article');

    $authors = $blog->authors()->getAll();

    $specificAuthor = $blog->authors()->loadFromSlug('john-smith');

    $comment = $blog->comments()->postComment(
        $specificArticle->getId(),
        'John Smith',
        new EmailAddress('john-smith@gmail.com'),
        'This is a comment.'
    );
}
```