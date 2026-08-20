# Building an adapter with adapter-common

The `cache/adapter-common` package implements the public PSR-6 and PSR-16 APIs. A storage adapter extends `AbstractCachePool` and implements a small set of backend hooks.

## Installation

```bash
composer require cache/adapter-common:^3.0
```

Version 3 requires PHP 8.2 and the v3 PSR cache interfaces.

## Upgrading a custom adapter

### Version 3

Version 3 stores the current generation for every item tag. `storeItemInCache()` must persist the list of `[tag, generation]` pairs from `PhpCacheItem::getTagVersions()`. `fetchObjectFromCache()` must return that same list.

The default implementation stores generation markers through the normal item hooks under `tagv!` keys. It stores tag indexes under `tag!` keys.

Each metadata key contains a SHA-256 digest of the tag. The key stays within the portable 64-character PSR-6 key alphabet. Both prefixes are now reserved for internal metadata.

Version 3 also adds the protected `appendListItemWithExpiration()` hook. The default implementation calls `appendListItem()` and ignores the expiration timestamp.

Custom subclasses that already declare `appendListItemWithExpiration()` must use this signature:

```php
protected function appendListItemWithExpiration(string $name, string $key, ?int $expirationTimestamp): bool;
```

The version 3 payload and marker format is incompatible with version 2 workers. Stop all workers and clear the backend before an upgrade or rollback.

### Version 2

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

Return `[false, null, [], null]` for a miss, an expired record, or malformed storage data. Each tag entry must be a two-element `[tag, generation]` string pair. The expiration value must be a Unix timestamp or `null`.

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

Override `appendListItemWithExpiration()` when the backend can expire a tag index with its items. The third argument is the item's absolute Unix expiration timestamp or `null`.

## Tag generation hooks

The default implementation stores generation markers through the normal item hooks. An adapter can override these protected methods when its backend has a better native representation:

```php
protected function readTagVersion(string $name): ?string;
protected function writeTagVersion(string $name, string $version): bool;
protected function deleteTagVersion(string $name): bool;
```

`getItem()` validates the stored `[tag, generation]` pairs automatically. An optimized read path that bypasses `getItem()` must call `tagVersionsAreCurrent()` before it returns a stored value.

## Error handling

Return `false` when a backend operation fails normally. Let backend exceptions reach `AbstractCachePool`, which applies the configured logger and PSR behavior.

Do not deserialize untrusted payloads without validating their shape and value types.

## Verification

Run the shared [integration test suites](integration-tests.md) against every supported backend and PHP version.
