# Namespaced PSR-6 cache pool 

The namespaced cache pool is a special cache of the hierarchical cache architecture where it only uses one level
of hierarchy. Use this when you provide a third party library with a PSR-6 cache pool to make sure you never
get key conflicts or unexpected calls to `CacheItemPoolInterface::clear()`. 
 
```php
// Get a pool that supports hierarchy
$client = new \Redis();
$client->connect('127.0.0.1', 6379);
$pool = new RedisCachePool($client);

// Decorate it with a NamesapcedPool
$namespacedPool = new NamespacedCachePool($pool, 'acme');

$item = $namespacedPool->getItem('foo')->set('bar');
$namespacedPool->save($item);

$namespacedPool->hasItem('foo'); // True
$pool->hasItem('foo'); // False
```

Internally the `NamespacedCachePool` will prepend all keys with `|acme|`. Which means that the `$pool` 
may access items in the namespace by doing the very same: 

```php
$pool->hasItem('|acme|foo'); // True
```

## Symfony integration

If you are are using our bundles you may configure a service with as a namespaced pool by using the `pool_namespace` 
configuration option. 

