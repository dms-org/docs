Blog Package
============

The blog package provides the functionality for building a simple blogging platform.

## Installation

Install the package via composer:

`composer require dms/package.blog`

Configure the blog package in `app/Providers/AppServiceProvider.php`

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