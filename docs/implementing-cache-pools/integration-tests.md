# Cache integration tests

The `cache/integration-tests` package verifies PSR-6, PSR-16, tag, and hierarchy behavior. Version 1 requires PHP 8.2, PHPUnit 11, the v3 PSR cache interfaces, and `cache/tag-interop` 2.

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
