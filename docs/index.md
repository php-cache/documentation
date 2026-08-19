# PHP Cache

![PHP Cache logo](https://raw.githubusercontent.com/php-cache/documentation/master/logos/php-cache-logo-256.png)

PHP Cache provides small, interoperable caching packages for PHP. Version 2 requires PHP 8.2. PSR-6 adapters use `psr/cache` 3, and PSR-16 implementations support `psr/simple-cache` 2 and 3.

Start with the [PSR cache introduction](introduction.md) if you are new to PSR-6 or PSR-16. Cache implementers can use the [integration test suite](implementing-cache-pools/integration-tests.md).

## Cache adapters

Each adapter is available as a separate Composer package. All PHP Cache adapters support tags. Some also support hierarchical keys.

| Adapter | Composer package | Hierarchy |
| --- | --- | --- |
| [APCu](https://github.com/php-cache/apcu-adapter) | `cache/apcu-adapter` | No |
| [Array](https://github.com/php-cache/array-adapter) | `cache/array-adapter` | Yes |
| [Chain](https://github.com/php-cache/chain-adapter) | `cache/chain-adapter` | No |
| [Doctrine](https://github.com/php-cache/doctrine-adapter) | `cache/doctrine-adapter` | No |
| [Filesystem](https://github.com/php-cache/filesystem-adapter) | `cache/filesystem-adapter` | No |
| [Illuminate](https://github.com/php-cache/illuminate-adapter) | `cache/illuminate-adapter` | Yes |
| [Memcache](https://github.com/php-cache/memcache-adapter) | `cache/memcache-adapter` | No |
| [Memcached](https://github.com/php-cache/memcached-adapter) | `cache/memcached-adapter` | Yes |
| [MongoDB](https://github.com/php-cache/mongodb-adapter) | `cache/mongodb-adapter` | No |
| [Predis](https://github.com/php-cache/predis-adapter) | `cache/predis-adapter` | Yes |
| [Redis](https://github.com/php-cache/redis-adapter) | `cache/redis-adapter` | Yes |
| [Void](https://github.com/php-cache/void-adapter) | `cache/void-adapter` | Yes |

Install only the adapter your app needs:

```bash
composer require cache/redis-adapter:^2.0
```

The `cache/cache` metapackage installs the complete adapter collection:

```bash
composer require cache/cache:^2.0
```

Some adapters also require a PHP extension or client library. Check the adapter README before installation.

## Upgrading to version 2

Do not run version 2 workers alongside older workers when they share an affected cache store.

Version 2 changes these internal formats:

* APCu stores native arrays instead of serialized strings.
* Redis and Predis store tag indexes as sets instead of lists.
* Prefix and namespace components use reversible `_xHH_` byte encoding. Bytes outside `[A-Za-z0-9_.]` and literal lowercase `_x` sequences are transformed.
* Namespaced public key components preserve ordinary backend-supported bytes, including `-`, `%`, and non-ASCII text. Only `|`, `!`, and literal lowercase `_x` sequences are transformed.
* Namespaced tag indexes are isolated in version 2. Clear namespaced caches containing tagged items before upgrading or rolling back.
* Public hierarchy keys use a separate storage path. Clear namespaced caches containing hierarchy keys before upgrading or rolling back.

Clear a namespaced cache before upgrading or rolling back when a namespace contains bytes outside `[A-Za-z0-9_.]` or a lowercase `_x` sequence. Clear it when a public key contains `|`, `!`, or a lowercase `_x` sequence.

Clear a prefixed cache when its prefix contains bytes outside `[A-Za-z0-9_.]` or a lowercase `_x` sequence.

Generic PSR-6 namespace decorators persist a random generation before deriving storage keys. They probe alternate metadata keys instead of overwriting an unrelated value.

Use this deployment sequence:

1. Stop or drain every old worker.
2. Clear each affected APCu, Redis, Predis, namespaced, or prefixed cache.
3. Deploy version 2 and restart the workers.

Use the same sequence before rolling back. No other cache formats change unless an adapter README says otherwise.

Update these client libraries before installing version 2:

* The Filesystem adapter supports Flysystem 2 and 3. It no longer supports Flysystem 1.
* The Illuminate adapter supports `illuminate/cache` 11 through 13.
* The Predis adapter supports Predis 2 and 3. It no longer supports Predis 1.

Custom adapters that extend `AbstractCachePool` must add version 2's native types. Follow the [custom adapter upgrade guide](implementing-cache-pools/adapter-common.md#upgrading-a-custom-adapter).

## Tags

Tags let an app invalidate related items without knowing every cache key.

```php
$product = $pool->getItem('product.42');
$product->set(['name' => 'Desk'])->setTags(['products', 'featured']);
$pool->save($product);

$pool->invalidateTag('products');

$pool->getItem('product.42')->isHit();
```

The final call returns `false` after invalidation. Pools and items expose tags through the interfaces in `cache/tag-interop`.

## Hierarchical keys

A hierarchical key starts with `|`. Deleting a parent path invalidates every cached descendant.

```php
$item = $pool->getItem('|users|42|followers|7|likes');
$item->set(12);
$pool->save($item);

$pool->deleteItem('|users|42|followers');

$pool->hasItem('|users|42|followers|7|likes');
```

The final call returns `false`. Read the [hierarchy guide](hierarchy.md) for details.

## Namespaces and prefixes

`NamespacedCachePool` isolates one logical cache inside any PSR-6 pool. Its `clear()` method invalidates only that namespace.

`PrefixedCachePool` works with any PSR-6 pool. Its `clear()` method clears the entire wrapped pool. Read the [namespace guide](namespace.md) before choosing between them.

Use each decorator's `create()` factory when the wrapped pool supports tags. The factory preserves taggable items and forwards tag invalidation.

## Chain pools

`CachePoolChain` reads through a list of PHP Cache pools and backfills earlier pools after a hit. Writes and tag invalidations run against each active pool.

The chain implements both PSR-6 and PSR-16. Each member must implement `Cache\Adapter\Common\PhpCachePool` so the chain can transfer items during backfills.

By default, a backend exception stops the operation. Set `skip_on_failure` to remove that pool from the current chain instance and continue:

```bash
composer require cache/chain-adapter:^2.0 cache/void-adapter:^2.0
```

```php
use Cache\Adapter\Chain\CachePoolChain;
use Cache\Adapter\Void\VoidCachePool;

$cache = new CachePoolChain(
    [$redisPool, new VoidCachePool()],
    ['skip_on_failure' => true],
);

$cache->setMultiple(['report' => $report], 60);
$values = $cache->getMultiple(['report', 'missing'], null);
```

Invalid cache keys still throw. `skip_on_failure` handles exceptions during cache operations, but it cannot handle a pool that fails during construction.

## Adapter-specific APIs

`MemcachedCachePool` sends PSR-16 bulk calls through Memcached's native `getMulti()`, `setMulti()`, and `deleteMulti()` commands. Bulk writes share one expiration and remove old tag references only after storage succeeds.

`FilesystemCachePool` exposes its Flysystem instance and cache folder through accessors:

```php
$pool->setFolder('tenant/cache');
$pool->setFilesystem($replacementFilesystem);

$folder = $pool->getFolder();
$filesystem = $pool->getFilesystem();
```

`setFolder()` normalizes slash styles and removes empty or current-directory segments. It rejects root and parent-directory paths. `setFilesystem()` creates the current cache directory on the replacement filesystem.

## Session locking

The PSR-6 and PSR-16 session handlers require a `SessionLockInterface`. Pass the lock before the options array:

```php
use Cache\SessionHandler\Psr6SessionHandler;

$handler = new Psr6SessionHandler($pool, $sessionLock, [
    'prefix' => 'session.',
    'ttl' => 3600,
]);
```

The lock must acquire each session ID atomically across every process that shares the cache. A cache read followed by a cache write is not an atomic lock.

The handler acquires the lock during session validation or reading. It holds the lock until PHP closes or destroys the session.

CacheBundle supplies a Symfony Lock implementation. Standalone users must provide an implementation with a lease that recovers from crashed requests.

## Bridges and integrations

The project also maintains these packages:

* [PSR-6 to Doctrine Cache bridge](doctrine-bridge.md)
* [PSR-6 to PSR-16 bridge](https://github.com/php-cache/simple-cache-bridge)
* [Encrypted cache decorator](https://github.com/php-cache/encryption-cache)
* [Taggable cache decorator](https://github.com/php-cache/taggable-cache)
* [PSR cache session handlers](https://github.com/php-cache/session-handler)
* [Symfony AdapterBundle](symfony/adapter-bundle.md)
* [Symfony CacheBundle](symfony/cache-bundle.md)

Report documentation or package problems on the [GitHub issue tracker](https://github.com/php-cache/cache/issues).
