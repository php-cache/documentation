# Documentation for PHP Cache

PHP Cache is an organisation that thinks that cache should be PSR-6 and lightweight. 

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

#### Hierarchy

Think of a hierarchy like a file system. If you remove a folder "Foo", all items and folders in "Foo" will also be removed. A hierarchical cache key must start with a pipe ("|").


```php
$pool->hasItem('|users|4711|followers|12|likes'); // True
$pool->deleteItem('|users|4711|followers');
$pool->hasItem('|users|4711|followers|12|likes'); // False
```

## Cache pool implementations

There are plenty of adapters in this organisaion. Each of them lives in a different repository. Splitting them up in multiple packages complies with the *Common reuse principle* and makes it easier for the developer to follow the changes of a specific adapter. 

Each adapter has it own features. The table below lists all our adapters and their features. 


| Adapter | Tagging | Hierarcy | Removes stale* | Badges |
| ------- | ------- | -------- | --------------- | ------ |
| [Apc] | Yes | No  | No | [![Latest Stable Version](https://poser.pugx.org/cache/apc-adapter/v/stable)](https://packagist.org/packages/cache/apc-adapter) [![Total Downloads](https://poser.pugx.org/cache/apc-adapter/downloads)](https://packagist.org/packages/cache/apc-adapter)
| [Apcu] | Yes | No | No | [![Latest Stable Version](https://poser.pugx.org/cache/apcu-adapter/v/stable)](https://packagist.org/packages/cache/apcu-adapter) [![Total Downloads](https://poser.pugx.org/cache/apcu-adapter/downloads)](https://packagist.org/packages/cache/apcu-adapter)
| [Array] | Yes | No | No | [![Latest Stable Version](https://poser.pugx.org/cache/array-adapter/v/stable)](https://packagist.org/packages/cache/array-adapter) [![Total Downloads](https://poser.pugx.org/cache/array-adapter/downloads)](https://packagist.org/packages/cache/array-adapter)
| Couchbase (via [Doctrine])| Yes | No |  | 
| [Filesystem](https://github.com/php-cache/filesystem-adapter) (via [Flysystem]) | Yes | No | No | [![Latest Stable Version](https://poser.pugx.org/cache/filesystem-adapter/v/stable)](https://packagist.org/packages/cache/filesystem-adapter) [![Total Downloads](https://poser.pugx.org/cache/filesystem-adapter/downloads)](https://packagist.org/packages/cache/filesystem-adapter)
| [Memcache] | Yes | No | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/memcache-adapter/v/stable)](https://packagist.org/packages/cache/memcache-adapter) [![Total Downloads](https://poser.pugx.org/cache/memcache-adapter/downloads)](https://packagist.org/packages/cache/memcache-adapter)
| [Memcached] | Yes | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/memcached-adapter/v/stable)](https://packagist.org/packages/cache/memcached-adapter) [![Total Downloads](https://poser.pugx.org/cache/memcached-adapter/downloads)](https://packagist.org/packages/cache/memcached-adapter)
| [MongoDB] | Yes | No | No | [![Latest Stable Version](https://poser.pugx.org/cache/mongodb-adapter/v/stable)](https://packagist.org/packages/cache/mongodb-adapter) [![Total Downloads](https://poser.pugx.org/cache/mongodb-adapter/downloads)](https://packagist.org/packages/cache/mongodb-adapter)
| [Predis]| Yes | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/predis-adapter/v/stable)](https://packagist.org/packages/cache/predis-adapter) [![Total Downloads](https://poser.pugx.org/cache/predis-adapter/downloads)](https://packagist.org/packages/cache/predis-adapter)
| [Redis] | Yes | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/redis-adapter/v/stable)](https://packagist.org/packages/cache/redis-adapter) [![Total Downloads](https://poser.pugx.org/cache/redis-adapter/downloads)](https://packagist.org/packages/cache/redis-adapter)
| Riak (via [Doctrine])| Yes | No |  | 
| SQLite3 (via [Doctrine])| Yes | No | No | 
| [Void] | Yes | No | No | [![Latest Stable Version](https://poser.pugx.org/cache/void-adapter/v/stable)](https://packagist.org/packages/cache/void-adapter) [![Total Downloads](https://poser.pugx.org/cache/void-adapter/downloads)](https://packagist.org/packages/cache/void-adapter)
| WinCache (via [Doctrine])| Yes | No |  | 
| Xcache (via [Doctrine])| Yes | No |  | 
| ZendData (via [Doctrine])| Yes | No |  | 
| | | | | 
| [Chain] | Yes | | | [![Latest Stable Version](https://poser.pugx.org/cache/chain-adapter/v/stable)](https://packagist.org/packages/cache/chain-adapter) [![Total Downloads](https://poser.pugx.org/cache/chain-adapter/downloads)](https://packagist.org/packages/cache/chain-adapter)
| [Doctrine] | Yes | No | Depends |  [![Latest Stable Version](https://poser.pugx.org/cache/doctrine-adapter/v/stable)](https://packagist.org/packages/cache/doctrine-adapter) [![Total Downloads](https://poser.pugx.org/cache/doctrine-adapter/downloads)](https://packagist.org/packages/cache/doctrine-adapter)

\* *Tagging and hierarchy store lots of extra items in cache that are never actively removed. Some implementations of cache storages like Redis and Memcache will automatically remove these items when they're stale or no longer used. That is why tagging and hierarchy will work better on such cache storages.*
  


#### Chain adapter

We also have a chain adapter where you can chain multiple pool together. It is great if you have a fast storage with limited memory and a slower storage with loads of memory. 

### Doctrine adapter

The doctrine adapter is a PSR-6 adapter that wraps a `Doctrine\Common\Cache\Cache` object. With this adapter you can use storages like Riak and WinCache which currently do not have any PHP Cache adapters. 

## Installation

Use composer to install any of the adapters above. Some adapters may require configuration before they can be used. 
Refer to the adapter's Github page to see how they are configured. You could also use the Symfony [AdapterBundle] to 
configure the adapters. 

```bash
composer require cache/[any]-adapter
```

### Requirements

Unless other is specified, all adapters support PHP version `~5.5` and `~7.0.0`. Most adapters do also have requirements
on PHP extension. Like the Redis adapter requires `ext-redis`. 

## Framework integration

#### Symfony

There are have two Symfony bundles; [AdapterBundle] and [CacheBundle]. 

The AdapterBundle is used to configure and register a PSR-6 cache pool as a Symfony service. The  CacheBundle is used to integrate **any** PSR-6 cache service with the framework. It supports session cache, doctrine cache, validation cache and many more. 

## Organisation overview

Excluding our adapters, we have the following packages

| Name | Description | Badges |
| ---- | ----------- | ------ |
| [AdapterBundle] | Bundle to register adapters to services. | [![Latest Stable Version](https://poser.pugx.org/cache/adapter-bundle/v/stable)](https://packagist.org/packages/cache/adapter-bundle) [![Total Downloads](https://poser.pugx.org/cache/adapter-bundle/downloads)](https://packagist.org/packages/cache/adapter-bundle)
| [Adapter common] | The `AbstractCachePool` and `CacheItem` live here. | [![Latest Stable Version](https://poser.pugx.org/cache/adapter-common/v/stable)](https://packagist.org/packages/cache/adapter-common) [![Total Downloads](https://poser.pugx.org/cache/adapter-common/downloads)](https://packagist.org/packages/cache/adapter-common)
| [CacheBundle] | Bundle to integrate **any** PSR-6 service with the<br>Symfony framework. | [![Latest Stable Version](https://poser.pugx.org/cache/cache-bundle/v/stable)](https://packagist.org/packages/cache/cache-bundle) [![Total Downloads](https://poser.pugx.org/cache/cache-bundle/downloads)](https://packagist.org/packages/cache/cache-bundle)
| [Doctrine bride] | A bridge from PSR-6 to DoctrineCache | [![Latest Stable Version](https://poser.pugx.org/cache/psr-6-doctrine-bridge/v/stable)](https://packagist.org/packages/cache/psr-6-doctrine-bridge) [![Total Downloads](https://poser.pugx.org/cache/psr-6-doctrine-bridge/downloads)](https://packagist.org/packages/cache/psr-6-doctrine-bridge)
| [Hierarchical cache] | A trait and interface to support cache hierachy | [![Latest Stable Version](https://poser.pugx.org/cache/hierarchical-cache/v/stable)](https://packagist.org/packages/cache/hierarchical-cache) [![Total Downloads](https://poser.pugx.org/cache/hierarchical-cache/downloads)](https://packagist.org/packages/cache/hierarchical-cache)
| [Integration tests] | Used to verify **any** PSR-6 implementation | [![Latest Stable Version](https://poser.pugx.org/cache/integration-tests/v/stable)](https://packagist.org/packages/cache/integration-tests) [![Total Downloads](https://poser.pugx.org/cache/integration-tests/downloads)](https://packagist.org/packages/cache/integration-tests)
| [Session handler] | Implementation of `\SessionHandlerInterface` | [![Latest Stable Version](https://poser.pugx.org/cache/session-handler/v/stable)](https://packagist.org/packages/cache/session-handler) [![Total Downloads](https://poser.pugx.org/cache/session-handler/downloads)](https://packagist.org/packages/cache/session-handler)
| [Taggable cache] | Traits and interfaces to support cache tagging | [![Latest Stable Version](https://poser.pugx.org/cache/taggable-cache/v/stable)](https://packagist.org/packages/cache/taggable-cache) [![Total Downloads](https://poser.pugx.org/cache/taggable-cache/downloads)](https://packagist.org/packages/cache/taggable-cache)

## Contact

[![Gitter](https://badges.gitter.im/php-cache/cache.svg)](https://gitter.im/php-cache/cache) 

We would love to hear form you. Ping us on twitter [@aequasi](https://twitter.com/aequasi) and [@tobiasnyholm](https://twitter.com/tobiasnyholm). You could also join us on [Gitter](https://gitter.im/php-cache/cache).

[AdapterBundle]: https://github.com/php-cache/adapter-bundle
[Adapter common]: https://github.com/php-cache/adapter-common
[Apc]: https://github.com/php-cache/apc-adapter
[Apcu]: https://github.com/php-cache/apcu-adapter
[Array]: https://github.com/php-cache/array-adapter
[CacheBundle]: https://github.com/php-cache/cache-bundle
[Chain]: https://github.com/php-cache/chain-adapter
[Doctrine]: https://github.com/php-cache/doctrine-adapter
[Doctrine bride]: https://github.com/php-cache/doctrine-bridge
[Filesystem]: https://github.com/php-cache/filesystem-adapter
[Hierarchical cache]: https://github.com/php-cache/hierarchical-cache
[Integration tests]: https://github.com/php-cache/integration-tests
[Memcache]: https://github.com/php-cache/memcache-adapter
[Memcached]: https://github.com/php-cache/memcached-adapter
[MongoDB]: https://github.com/php-cache/mongodb-adapter
[Predis]: https://github.com/php-cache/predis-adapter
[Redis]: https://github.com/php-cache/redis-adapter
[Session handler]: https://github.com/php-cache/session-handler
[Taggable cache]: https://github.com/php-cache/taggable-cache
[Void]: https://github.com/php-cache/void-adapter
[Flysystem]: http://flysystem.thephpleague.com/
