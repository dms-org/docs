Repositories
============

Persisting entities
-------------------

You can save (create or update) entities to the database using the repository 

```php
<?php

function saveItem(ITodoItemRepository $repo)
{
    $repo->save(new TodoItem('some item'));
    
    $repo->saveAll([
        new TodoItem('an item'),
        new TodoItem('another item'),
    ]);
}
```

Loading entities
----------------

You can load entities from the database

```php
<?php

function getItem(ITodoItemRepository $repo)
{
    // Get the entity with the id = 1, throw if not found
    $entity = $repo->get(1);
    
    // Get the entity with the id = 1, null if not found
    $entity = $repo->tryGet(1);
    
    // Get the entities with the ids 1, 2 or 3, throw if any not found
    $entities = $repo->getAll([1, 2, 3]);
    
    // Get the entity with the ids 1, 2 or 3, ignore if any not found
    $entities = $repo->tryGetAll([1, 2, 3]);
}
```

Deleting entities
-----------------

You can load entities from the database

```php
<?php

function deleteItems(ITodoItemRepository $repo)
{
    // Delete the entity
    $repo->remove($someEntity);
    
    // Delete the entity with the ids 1, 2 or 3
    $repo->removeById(1);
    
    // Delete the entity
    $repo->removeAll([$someEntity, $anotherEntity]);
    
    // Delete the entity with the ids 1, 2 or 3
    $repo->removeAllById([1, 2, 3]);
    
    // Delete all entities
    $repo->clear();
}
```

Criteria
--------

Simple queries can be expressed through criteria, so you don't need to resort
to SQL for most of your database operations.

```php
<?php

function getEntitiesWithCriteria(ITodoItemRepository $repo)
{
    $items = $repo->matching(
        $repo->criteria()
            ->where(TodoItem::COMPLETED, '=', true)
    );

    $items = $repo->matching(
        $repo->criteria()
            ->whereStringContainsCaseInsensitive(TodoItem::DESCRIPTION, 'some text')
            ->orderByDesc(TodoItem::ID)
            ->skip(5)
            ->limit(10)
    );
}

function removeEntitiesWithCriteria(ITodoItemRepository $repo)
{
    $repo->removeMatching(
        $repo->criteria()
            ->whereIn(TodoItem::ID, [1, 2, 3])
    );
}
```

Custom Queries
--------------

Add methods to the repository for custom database queries.

```php
<?php declare(strict_types = 1);

namespace App\Infrastructure\Persistence;

use Dms\Core\Persistence\Db\Connection\IConnection;
use Dms\Core\Persistence\Db\Mapping\IOrm;
use Dms\Core\Persistence\DbRepository;
use App\Domain\Services\Persistence\ITodoItemRepository;
use App\Domain\Entities\TodoItem;

/**
 * The database repository implementation for the App\Domain\Entities\TodoItem entity.
 */
class DbTodoItemRepository extends DbRepository implements ITodoItemRepository
{
    public function __construct(IConnection $connection, IOrm $orm)
    {
        parent::__construct($connection, $orm->getEntityMapper(TodoItem::class));
    }

    /**
     * Gets five random todo items
     * 
     * @return TodoItem[]
     */
    public function loadRandomItems() : array
    {
        return $this->loadQuery('
            SELECT * FROM todo_items
            ORDER BY RAND()
            LIMIT :limit
        ', ['limit' => 5]);
    }

    /**
     * Marks all todo items as complete
     * 
     * @return void
     */
    public function markAllItemsAsComplete()
    {
        $this->connection->prepare('
            UPDATE todo_items
            SET completed = TRUE
        ')->execute();
    }
}
```
