Custom Actions
==============

It is often necessary to support add functionality to your CMS that is not part of the standard CRUD operations.
You can define custom actions in your modules which can be executed in the backend.

## General Actions

For actions that operate on multiple entities you can define a general action as follows


```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Common\Structure\FileSystem\File;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IArticleRepository;
use App\Domain\Entities\Article;
use App\Domain\Services\ArticleExportService;
use Dms\Common\Structure\Field;
use Dms\Core\Form\Builder\Form;

/**
 * The article module.
 */
class ArticleModule extends CrudModule
{
    /**
     * @var ArticleExportService
     */ 
    public $articleExportService;

    public function __construct(IArticleRepository $dataSource, IAuthSystem $authSystem, ArticleExportService $articleExportService)
    {
        $this->articleExportService = $articleExportService;
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

        $module->action('export')
            ->form(Form::create()->section('Settings', [
                Field::create('date_range', 'Date Range')->dateRange()->required()
            ]))
            ->returns(File::class)
            ->handler(function (array $input) {
                // Generate your export file...
                return $this->articleExportService->generateExportForDateRange($input['date_range']);
            });

        // Omitted the rest of the module configuration
    }
}
```

## Object Actions

If the action operates on one entity, you can define an object action

```php
<?php declare(strict_types = 1);

namespace App\Cms\Modules;

use Dms\Common\Structure\FileSystem\File;
use Dms\Core\Auth\IAuthSystem;
use Dms\Core\Common\Crud\CrudModule;
use Dms\Core\Common\Crud\Definition\CrudModuleDefinition;
use Dms\Core\Common\Crud\Definition\Form\CrudFormDefinition;
use Dms\Core\Common\Crud\Definition\Table\SummaryTableDefinition;
use App\Domain\Services\Persistence\IJobRepository;
use App\Domain\Entities\Job;
use Dms\Common\Structure\Field;
use Dms\Core\Form\Builder\Form;

/**
 * The job module.
 */
class JobModule extends CrudModule
{
    /**
     * @var JobCompletionService
     */
    protected $jobCompletionService;

    public function __construct(IJobRepository $dataSource, IAuthSystem $authSystem, JobCompletionService $jobCompletionService)
    {
        $this->jobCompletionService = $jobCompletionService;
        parent::__construct($dataSource, $authSystem);
    }

    /**
     * Defines the structure of this module.
     *
     * @param CrudModuleDefinition $module
     */
    protected function defineCrudModule(CrudModuleDefinition $module)
    {
        $module->name('job');

        $module->labelObjects()->fromProperty(Job::NAME);

        $module->objectAction('mark-as-complete')
            ->form(Form::create()->section('Details', [
                Field::create('comments', 'Comments')->string()
            ]))
            ->handler(function (Job $job, array $input) {
                $this->jobCompletionService->completeJob($job, $input['comments']);
            });

        // Omitted module configuration
    }
}
```