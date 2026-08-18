# Introduction to PSR caching

[PSR-6](https://www.php-fig.org/psr/psr-6/) defines cache items and cache pools. [PSR-16](https://www.php-fig.org/psr/psr-16/) defines a smaller key and value API called Simple Cache.

PHP Cache version 2 implements version 3 of both interface packages and requires PHP 8.2.

## PSR-6 basics

Apps request items from a pool. The pool creates new items on a miss, so app code should never construct cache items directly.

```php
use Cache\Adapter\PHPArray\ArrayCachePool;

$pool = new ArrayCachePool();
$item = $pool->getItem('report.current');

if (!$item->isHit()) {
    $item->set(buildCurrentReport());
    $item->expiresAfter(60);
    $pool->save($item);
}

$report = $item->get();
```

A cache item belongs to the pool that created it. Do not pass an item to another pool's `save()` or `saveDeferred()` method.

## PSR-16 basics

PHP Cache adapters also expose the PSR-16 `CacheInterface`:

```php
$report = $cache->get('report.current');

if (null === $report) {
    $report = buildCurrentReport();
    $cache->set('report.current', $report, 60);
}
```

Use a distinct sentinel as the default when `null` is a valid cached value.

## Portable cache keys

PSR-6 and PSR-16 implementations must support letters, numbers, underscores, and periods. The characters `{ } ( ) / \\ @ :` are reserved.

Implementations may accept other characters, but portable applications should stay within the required set. Hash long or user-controlled input before using it as a key.

Numeric-looking strings, such as `'123'`, are valid keys. Batch reads preserve the string key while you iterate their returned `iterable`.

Converting that `iterable` to an array may turn the key into an integer under PHP's array-key rules.

```php
$cacheKey = 'page.'.hash('sha256', $_SERVER['REQUEST_URI']);
```

## Choosing an adapter

The [adapter table](index.md#cache-adapters) shows available backends and hierarchy support.

* Use APCu for a fast shared-memory cache on one host.
* Use Array for tests and short-lived scripts.
* Use Memcached for a distributed volatile cache.
* Use Redis or Predis when you need shared storage and hierarchical keys.

Each adapter README documents its client setup and extra runtime requirements.
