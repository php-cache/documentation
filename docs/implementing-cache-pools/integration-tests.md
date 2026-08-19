# Cache integration tests

The `cache/integration-tests` package verifies PSR-6, PSR-16, tag, and hierarchy behavior. Version 1 requires PHP 8.2, PHPUnit 11.5 or 12, `psr/cache` 3, `psr/simple-cache` 2 or 3, and `cache/tag-interop` 2.

## Installation

```bash
composer require --dev cache/integration-tests:^1.0
```

Use a compatible PHPUnit version from the package's Composer constraints.

## Test a PSR-6 pool

```php
use Cache\IntegrationTests\CachePoolTest;
use Psr\Cache\CacheItemPoolInterface;

final class PoolIntegrationTest extends CachePoolTest
{
    public function createCachePool(): CacheItemPoolInterface
    {
        return new MyCachePool();
    }
}
```

Return a fresh pool for each test setup. The suite clears it during teardown.

## Test a PSR-16 cache

```php
use Cache\IntegrationTests\SimpleCacheTest;
use Psr\SimpleCache\CacheInterface;

final class SimpleCacheIntegrationTest extends SimpleCacheTest
{
    public function createSimpleCache(): CacheInterface
    {
        return new MySimpleCache();
    }
}
```

Override `advanceTime()` when the backend offers a test clock. This avoids real sleeps in expiration tests.

## Test tags

```php
use Cache\IntegrationTests\TaggableCachePoolTest;
use Cache\TagInterop\TaggableCacheItemPoolInterface;

final class TagIntegrationTest extends TaggableCachePoolTest
{
    public function createCachePool(): TaggableCacheItemPoolInterface
    {
        return new MyTaggableCachePool();
    }
}
```

## Test hierarchical keys

```php
use Cache\IntegrationTests\HierarchicalCachePoolTest;
use Psr\Cache\CacheItemPoolInterface;

final class HierarchyIntegrationTest extends HierarchicalCachePoolTest
{
    public function createCachePool(): CacheItemPoolInterface
    {
        return new MyHierarchicalCachePool();
    }
}
```

## Skip unsupported behavior

Each base class exposes a typed map of test names to reasons. Skip tests only for documented backend limitations.

```php
use Cache\IntegrationTests\CachePoolTest;
use Psr\Cache\CacheItemPoolInterface;

final class PoolWithSkippedExpirationTest extends CachePoolTest
{
    /** @var array<string, string> */
    protected array $skippedTests = [
        'testExpiration' => 'This backend does not support expiration.',
    ];

    public function createCachePool(): CacheItemPoolInterface
    {
        return new MyCachePool();
    }
}
```

Do not use `dev-master`. Pin the stable major so new breaking test APIs arrive only in a deliberate dependency upgrade.

## PSR-6 interpretation notes

The [PSR-6 specification](https://www.php-fig.org/psr/psr-6/) leaves a few edge cases open to interpretation. The integration suite applies these rules consistently:

* `getItems()` yields each item under its public cache key. Numeric string keys remain strings.
* Saving an item whose expiration has passed leaves that key as a miss. The suite does not require a specific `save()` return value for this case.
* A deferred item is visible to the same pool before `commit()`. Deleting the item or clearing the pool must prevent a later commit from restoring it.
* Bulk operations validate every key before changing the cache. An invalid key must not leave a partially applied deletion.

These rules describe the PHP Cache test suite. They do not add requirements to PSR-6 itself. See the corresponding tests in [`CachePoolTest`](https://github.com/php-cache/integration-tests/blob/1.x/src/CachePoolTest.php).
