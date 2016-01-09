# Documentation for PHP Cache

PHP Cahe is an organisation that thinks that cache should be PSR-6 and light weight. 

## Features

#### Tagging

Tags can be used to separate the storage of different systems in the cache. This allows different sections to be cleared on an individual level, while also preventing overlapping keys.

```php
$item = $pool->getItem('bow', ['weapons'])->set('weapon_value');
$pool->save($item);

$item = $pool->getItem('bow', ['london_districts'])->set('london_value');
$pool->save($item);

$pool->getItem('bow', ['weapons'])->isHit(); // true
$pool->getItem('bow', ['london_districts'])->isHit(); // true

$pool->clear(['london_districts']);
$pool->getItem('bow', ['weapons'])->isHit(); // true
$pool->getItem('bow', ['london_districts'])->isHit(); // false
```

#### Hierachy

Think of a hierarchy like a file system. If you remove a folder A, all items and folder in A will also be removed. A hierachy cache key must start with pipe ("|").


```php
$pool->hasItem('|users|4711|followers|12|likes'); // True
$pool->deleteItem('|users|4711|followers');
$pool->hasItem('|users|4711|followers|12|likes'); // False
```

#### Removes stalled items

When you do tagging or hierachy we store lots of items in cache that are never removed. Some implementations of cache (not adapters nor pools) like Redis and Memcache will automatically remove these when they stalled or not longer used. That is why tagging and hierarchy will work better on such implementation.  


## Cache pool implementations
There are plenty of adapters in this organisaion. Each of them lives in a different repository. Splitting them up in multiple packages complies with the *Common resue principle* and makes it easier for the developer to follow the changes of a specific adapter. 

Each adapter has it own features. The table below list all our adapters and their features. 


| Adapter | Tagging | Hierarcy | Removes stalled | Badges |
| ------- | ------- | -------- | --------------- | ------ |
| [Apc] | Yes | No  | No |
| [Apcu] | Yes | No | No |
| [Array] | Yes | No | No |
| Couchbase (via [Doctrine])| Yes | No |  |
| [Filesystem] (via [Flysystem]) | Yes | No | No |
| [Memcache] | Yes | No | Yes |
| [Memcached] | Yes | No | Yes |
| [MongoDB] | Yes | No | No |
| [Predis]| Yes | Yes | Yes |
| [Redis] | Yes | No | Yes |
| Riak (via [Doctrine])| Yes | No |  |
| SQLite3 (via [Doctrine])| Yes | No | No |
| [Void| Yes | No | No |
| WinCache (via [Doctrine])| Yes | No |  |
| Xcache (via [Doctrine])| Yes | No |  |
| ZendData (via [Doctrine])| Yes | No |  |

#### Chain adapter

We also have a chain adapter where you can chain multiple pool together. It is great if you got a fast storage with limited memory and a slower storage with loads of memory. 

## Framework integration

#### Symfony

There are have two Symfony bundles; [AdapterBundle](https://github.com/php-cache/adapter-bundle) and [CacheBundle](https://github.com/php-cache/cache-bundle). 

The AdapterBundle is used to configure and register a PSR-6 cache pool as a Symfony service. The  [CacheBundle](https://github.com/php-cache/cache-bundle) is used to integrate *any* PSR-6 cache service with the framework. It supports session cache, doctrine cache, validation cache and many more. 


[Apc]: https://github.com/php-cache/apc-adapter
[Apcu]: https://github.com/php-cache/apcu-adapter
[Array]: https://github.com/php-cache/array-adapter
[Filesystem]: https://github.com/php-cache/filesystem-adapter
[Memcache]: https://github.com/php-cache/memcache-adapter
[Memcached]: https://github.com/php-cache/memcached-adapter
[MongoDB]: https://github.com/php-cache/mongodb-adapter
[Predis]: https://github.com/php-cache/rredis-adapter
[Redis]: https://github.com/php-cache/redis-adapter
[Void]: https://github.com/php-cache/void-adapter

[Doctrine]: https://github.com/php-cache/doctrine-adapter
[Flysystem]: http://flysystem.thephpleague.com/

