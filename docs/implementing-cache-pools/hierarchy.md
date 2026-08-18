# Implementing hierarchical cache pools

The `cache/hierarchical-cache` package provides `HierarchicalPoolInterface` and `HierarchicalCachePoolTrait`. The trait maps a public hierarchy path to an internal storage key.

Use a backend that can increment path records atomically. Automatic eviction of stale records also helps control storage growth.

## Add the interface and trait

```php
use Cache\Adapter\Common\AbstractCachePool;
use Cache\Hierarchy\HierarchicalCachePoolTrait;
use Cache\Hierarchy\HierarchicalPoolInterface;

abstract class MyCachePool extends AbstractCachePool implements HierarchicalPoolInterface
{
    use HierarchicalCachePoolTrait;
}
```

The class still needs the storage and tag hooks required by `AbstractCachePool`.

## Translate every item key

Call `getHierarchyKey()` before reading, writing, or deleting a stored item. Non-hierarchical keys pass through unchanged.

For adapters based on `AbstractCachePool`, translate keys in these hooks:

* `fetchObjectFromCache()`
* `storeItemInCache()`
* `clearOneObjectFromCache()`

For example, a store hook begins with this translation:

```php
use Cache\Adapter\Common\AbstractCachePool;
use Cache\Adapter\Common\PhpCacheItem;
use Cache\Hierarchy\HierarchicalCachePoolTrait;
use Cache\Hierarchy\HierarchicalPoolInterface;

abstract class MyCachePool extends AbstractCachePool implements HierarchicalPoolInterface
{
    use HierarchicalCachePoolTrait;

    protected function storeItemInCache(PhpCacheItem $item, ?int $ttl): bool
    {
        $storageKey = $this->getHierarchyKey($item->getKey());

        return $this->storage->write($storageKey, $item, $ttl);
    }
}
```

The example assumes the adapter's storage client provides `write()`.

## Read path records directly

The trait calls `getDirectValue()` when it resolves a hierarchy path. This method must read the raw backend key without calling `getHierarchyKey()` again.

```php
abstract class MyCachePool
{
    public function getDirectValue(string $name): mixed
    {
        return $this->storage->get($name);
    }
}
```

The required method name is `getDirectValue()`. Older documentation called it `getValueFormStore()`, but that method no longer exists.

## Invalidate a branch

Pass a second argument to `getHierarchyKey()` when deleting an item. The trait writes the final path record into that argument.

For a hierarchical key, atomically increment the path record. This changes the derived storage keys for the branch and every descendant.

```php
abstract class MyCachePool
{
    protected function clearOneObjectFromCache(string $key): bool
    {
        $pathKey = null;
        $storageKey = $this->getHierarchyKey($key, $pathKey);
        $generationAdvanced = true;

        if (null !== $pathKey) {
            $generationAdvanced = $this->storage->increment($pathKey);
        }

        $this->clearHierarchyKeyCache();
        $deleted = $this->storage->delete($storageKey);

        return $generationAdvanced && $deleted;
    }
}
```

The storage client must initialize a missing path record to a numeric value before incrementing it. A non-hierarchical key leaves `$pathKey` as `null`.

Return `false` when the backend cannot advance the path record. Otherwise, callers can receive a successful deletion while descendant entries remain reachable.

Call `clearHierarchyKeyCache()` after clearing the entire backend as well. Otherwise, one PHP process can keep stale derived keys in memory.

## Test the implementation

Extend `HierarchicalCachePoolTest` from `cache/integration-tests:^1.0`. Run it together with the PSR-6 and tagging suites.
