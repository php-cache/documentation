# Building an adapter with adapter-common

The `cache/adapter-common` package implements the public PSR-6 and PSR-16 APIs. A storage adapter extends `AbstractCachePool` and implements a small set of backend hooks.

## Installation

```bash
composer require cache/adapter-common:^2.0
```

Version 2 requires PHP 8.2 and the v3 PSR cache interfaces.

## Upgrading a custom adapter

Version 2 adds native parameter and return types to every abstract hook. A subclass with the old signatures fails during class loading.

Update all eight hook declarations to match the signatures below. If the adapter overrides inherited methods, match those methods against version 2's [`AbstractCachePool`](https://github.com/php-cache/cache/blob/master/src/Adapter/Common/AbstractCachePool.php).

## Item storage hooks

An adapter must implement these methods:

```php
use Cache\Adapter\Common\AbstractCachePool;
use Cache\Adapter\Common\PhpCacheItem;

abstract class StorageCachePool extends AbstractCachePool
{
    abstract protected function storeItemInCache(PhpCacheItem $item, ?int $ttl): bool;

    abstract protected function fetchObjectFromCache(string $key): array;

    abstract protected function clearAllObjectsFromCache(): bool;

    abstract protected function clearOneObjectFromCache(string $key): bool;
}
```

`storeItemInCache()` receives a relative TTL in seconds or `null` for no expiration.

`fetchObjectFromCache()` returns a four-element record:

```php
[true, $value, $tags, $expirationTimestamp];
```

Return `[false, null, [], null]` for a miss, an expired record, or malformed storage data. The tags must use strings for both keys and values. The expiration value must be a Unix timestamp or `null`.

## Tag index hooks

`AbstractCachePool` also requires four methods for its tag index:

```php
use Cache\Adapter\Common\AbstractCachePool;

abstract class TaggedStorageCachePool extends AbstractCachePool
{
    abstract protected function getList(string $name): array;

    abstract protected function removeList(string $name): bool;

    abstract protected function appendListItem(string $name, string $key): bool;

    abstract protected function removeListItem(string $name, string $key): bool;
}
```

`getList()` returns a list of cache keys. The other methods update that list atomically where the backend supports atomic operations.

Return `false` when the backend cannot update a tag index. `AbstractCachePool` reports that failure from the save, delete, or invalidation operation.

## Error handling

Return `false` when a backend operation fails normally. Let backend exceptions reach `AbstractCachePool`, which applies the configured logger and PSR behavior.

Do not deserialize untrusted payloads without validating their shape and value types.

## Verification

Run the shared [integration test suites](integration-tests.md) against every supported backend and PHP version.
