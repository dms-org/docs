Many to Many
===========

A many to many bi-directional relationship between entities can be defined as follows.

### Entity Structure

#### app/Domain/Entities/Article.php

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\EntityCollection;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Article extends Entity
{
    const TITLE = 'title';
    const TAGS = 'tags';

    /**
     * @var string
     */
    public $title;

    /**
     * @var EntityCollection|Tag[]
     */
    public $tags;

    /**
     * Article constructor.
     */
    public function __construct()
    {
        parent::__construct();
        $this->tags = Tag::collection();
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->title)->asString();

        $class->property($this->tags)->asType(Tag::collectionType());
    }
}
```

#### app/Domain/Entities/Tag.php

```php
<?php declare(strict_types=1);

namespace App\Domain\Entities;

use Dms\Core\Model\EntityCollection;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;

class Tag extends Entity
{
    const NAME = 'name';
    const ARTICLES = 'articles';

    /**
     * @var string
     */
    public $name;

    /**
     * @var EntityCollection|Article[]
     */
    public $articles;

    /**
     * Tag constructor.
     */
    public function __construct()
    {
        parent::__construct();
        $this->articles = Article::collection();
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->name)->asString();

        $class->property($this->articles)->asType(Article::collectionType());
    }
}
```


### Mapper Configuration

#### app/Infrastructure/Persistence/ArticleMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Article;
use App\Domain\Entities\Tag;

/**
 * The App\Domain\Entities\Article entity mapper.
 */
class ArticleMapper extends EntityMapper
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
        $map->type(Article::class);
        $map->toTable('articles');

        $map->idToPrimaryKey('id');

        $map->property(Article::TITLE)->to('title')->asVarchar(255);

        $map->relation(Article::TAGS)
            ->to(Tag::class)
            ->toMany()
            ->withBidirectionalRelation(Tag::ARTICLES)
            ->throughJoinTable('article_tags')
            ->withParentIdAs('article_id')
            ->withRelatedIdAs('tag_id');
    }
}
```

#### app/Infrastructure/Persistence/TagMapper.php

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Mapping\Definition\MapperDefinition;
use Dms\Core\Persistence\Db\Mapping\EntityMapper;
use App\Domain\Entities\Tag;
use App\Domain\Entities\Article;

/**
 * The App\Domain\Entities\Tag entity mapper.
 */
class TagMapper extends EntityMapper
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
        $map->type(Tag::class);
        $map->toTable('tags');

        $map->idToPrimaryKey('id');

        $map->property(Tag::NAME)->to('name')->asVarchar(255);

        $map->relation(Tag::ARTICLES)
            ->to(Article::class)
            ->toMany()
            ->withBidirectionalRelation(Article::TAGS)
            ->throughJoinTable('article_tags')
            ->withParentIdAs('tag_id')
            ->withRelatedIdAs('article_id');
    }
}
```

### Module Configuration

#### app/Cms/Modules/ArticleModule.php

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IArticleRepository;
use App\Domain\Entities\Article;
use Dms\Common\Structure\Field;
use App\Domain\Services\Persistence\ITagRepository;
use App\Domain\Entities\Tag;

/**
 * The article module.
 */
class ArticleModule extends CrudModule
{
    /**
     * @var ITagRepository
     */
    protected $tagRepository;

    public function __construct(IArticleRepository $dataSource, IAuthSystem $authSystem, ITagRepository $tagRepository)
    {
        $this->tagRepository = $tagRepository;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('article');

        $module->labelObjects()->fromProperty(Article::TITLE);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('title', 'Title')->string()->required()
                )->bindToProperty(Article::TITLE),
                //
                $form->field(
                    Field::create('tags', 'Tags')
                        ->entitiesFrom($this->tagRepository)
                        ->labelledBy(Tag::NAME)
                        ->mapToCollection(Tag::collectionType())
                        // Add this line if you want to load the
                        // options asynchronously via autocomplete
                        ->searchableBy(Tag::NAME)
                )->bindToProperty(Article::TAGS),
            ]);
        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Article::TITLE)->to(Field::create('title', 'Title')->string()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```

#### app/Cms/Modules/TagModule.php

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\ITagRepository;
use App\Domain\Entities\Tag;
use Dms\Common\Structure\Field;
use App\Domain\Services\Persistence\IArticleRepository;
use App\Domain\Entities\Article;

/**
 * The tag module.
 */
class TagModule extends CrudModule
{
    /**
     * @var IArticleRepository
     */
    protected $articleRepository;

    public function __construct(ITagRepository $dataSource, IAuthSystem $authSystem, IArticleRepository $articleRepository)
    {
        $this->articleRepository = $articleRepository;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('tag');

        $module->labelObjects()->fromProperty(Tag::NAME);

        $module->crudForm(function (CrudFormDefinition $form) {
            $form->section('Details', [
                $form->field(
                    Field::create('name', 'Name')->string()->required()
                )->bindToProperty(Tag::NAME),
                //
                $form->field(
                    Field::create('articles', 'Articles')
                        ->entitiesFrom($this->articleRepository)
                        ->labelledBy(Article::TITLE)
                        ->mapToCollection(Article::collectionType())
                        // Add this line if you want to load the
                        // options asynchronously via autocomplete
                        ->searchableBy(Article::TITLE)
                )->bindToProperty(Tag::ARTICLES),
                //
            ]);

        });

        $module->removeAction()->deleteFromDataSource();

        $module->summaryTable(function (SummaryTableDefinition $table) {
            $table->mapProperty(Tag::NAME)->to(Field::create('name', 'Name')->string()->required());

            $table->view('all', 'All')
                ->loadAll()
                ->asDefault();
        });
    }
}
```