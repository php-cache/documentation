# PHP Cache 

<img align="right" src="https://raw.githubusercontent.com/php-cache/documentation/master/logos/php-cache-logo-256.png" />


The PHP Cache organization is dedicated to providing solid, powerful, flexible, and lightweight caching libraries for PHP
projects. All of the adapters we have created are [PSR-6](http://www.php-fig.org/psr/psr-6/) and [PSR-16](http://www.php-fig.org/psr/psr-16/) 
compliant. If you are a library implementer, we even have a [repository of tests ](https://github.com/php-cache/integration-tests) 
to help you meet the PSR specification.

Below you will find information about what features our libraries offer, and what adapters we have. You can also find out
framework integration libraries.

If you are new to PSR-6 caching you may want to have a look at our [introduction](introduction.md).

## Cache pool implementations

There are plenty of adapters in this organisaion. Each of them lives in a different repository. Splitting them up in multiple
packages complies with the *Common reuse principle* and makes it easier for the developer to follow the changes of a specific
adapter. 

Each adapter has it own features. The table below lists all our adapters and their features. 


| Adapter | Tagging | Hierarchy* | Badges |
| ------- | ------- | ---------- | ------ |
| [Apc] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/apc-adapter/v/stable)](https://packagist.org/packages/cache/apc-adapter) [![Total Downloads](https://poser.pugx.org/cache/apc-adapter/downloads)](https://packagist.org/packages/cache/apc-adapter)
| [Apcu] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/apcu-adapter/v/stable)](https://packagist.org/packages/cache/apcu-adapter) [![Total Downloads](https://poser.pugx.org/cache/apcu-adapter/downloads)](https://packagist.org/packages/cache/apcu-adapter)
| [Array] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/array-adapter/v/stable)](https://packagist.org/packages/cache/array-adapter) [![Total Downloads](https://poser.pugx.org/cache/array-adapter/downloads)](https://packagist.org/packages/cache/array-adapter)
| Couchbase (via [Doctrine]) | Yes | No | | 
| [Filesystem](https://github.com/php-cache/filesystem-adapter) (via [Flysystem]) | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/filesystem-adapter/v/stable)](https://packagist.org/packages/cache/filesystem-adapter) [![Total Downloads](https://poser.pugx.org/cache/filesystem-adapter/downloads)](https://packagist.org/packages/cache/filesystem-adapter)
| [Illuminate] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/illuminate-adapter/v/stable)](https://packagist.org/packages/cache/illuminate-adapter) [![Total Downloads](https://poser.pugx.org/cache/illuminate-adapter/downloads)](https://packagist.org/packages/cache/illuminate-adapter)
| [Memcache] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/memcache-adapter/v/stable)](https://packagist.org/packages/cache/memcache-adapter) [![Total Downloads](https://poser.pugx.org/cache/memcache-adapter/downloads)](https://packagist.org/packages/cache/memcache-adapter)
| [Memcached] | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/memcached-adapter/v/stable)](https://packagist.org/packages/cache/memcached-adapter) [![Total Downloads](https://poser.pugx.org/cache/memcached-adapter/downloads)](https://packagist.org/packages/cache/memcached-adapter)
| [MongoDB] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/mongodb-adapter/v/stable)](https://packagist.org/packages/cache/mongodb-adapter) [![Total Downloads](https://poser.pugx.org/cache/mongodb-adapter/downloads)](https://packagist.org/packages/cache/mongodb-adapter)
| [Predis] | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/predis-adapter/v/stable)](https://packagist.org/packages/cache/predis-adapter) [![Total Downloads](https://poser.pugx.org/cache/predis-adapter/downloads)](https://packagist.org/packages/cache/predis-adapter)
| [Redis] | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/redis-adapter/v/stable)](https://packagist.org/packages/cache/redis-adapter) [![Total Downloads](https://poser.pugx.org/cache/redis-adapter/downloads)](https://packagist.org/packages/cache/redis-adapter)
| Riak (via [Doctrine])| Yes | No | | 
| SQLite3 (via [Doctrine])| Yes | No | | 
| [Void] | Yes | Yes | [![Latest Stable Version](https://poser.pugx.org/cache/void-adapter/v/stable)](https://packagist.org/packages/cache/void-adapter) [![Total Downloads](https://poser.pugx.org/cache/void-adapter/downloads)](https://packagist.org/packages/cache/void-adapter)
| WinCache (via [Doctrine]) | Yes | No | | 
| Xcache (via [Doctrine]) | Yes | No | | 
| ZendData (via [Doctrine]) | Yes | No | | 
| | | | | 
| [Chain] | Yes | | [![Latest Stable Version](https://poser.pugx.org/cache/chain-adapter/v/stable)](https://packagist.org/packages/cache/chain-adapter) [![Total Downloads](https://poser.pugx.org/cache/chain-adapter/downloads)](https://packagist.org/packages/cache/chain-adapter)
| [Doctrine] | Yes | No | [![Latest Stable Version](https://poser.pugx.org/cache/doctrine-adapter/v/stable)](https://packagist.org/packages/cache/doctrine-adapter) [![Total Downloads](https://poser.pugx.org/cache/doctrine-adapter/downloads)](https://packagist.org/packages/cache/doctrine-adapter)

\* *Hierarchy store lots of extra items in cache that are never actively removed. Some implementations of cache storages 
like Redis and Memcache will automatically remove these items when they're stale or no longer used. That is why hierarchy
will work better on such cache storages.*
  


### Chain adapter

We also have a chain adapter where you can chain multiple pool together. It is great if you have a fast storage with limited 
memory and a slower storage with loads of memory. 

### Doctrine adapter

The doctrine adapter is a PSR-6 adapter that wraps a `Doctrine\Common\Cache\Cache` object. With this adapter you can use 
storages like Riak and WinCache which currently do not have any PHP Cache adapters. 

## Installation

Use composer to install any of the adapters above. Some adapters may require configuration before they can be used. 
Refer to the adapter's Github page to see how they are configured. You could also use the Symfony [AdapterBundle] to 
configure the adapters. 

```bash
composer require cache/[any]-adapter
```

You can also install all of the adapters with the `cache/cache`

```bash
composer require cache/cache
```

### Requirements

Unless other is specified, all adapters support PHP version `^5.5` and `^7.0`. Most adapters do also have requirements
on PHP extension. Like the Redis adapter requires `ext-redis`. 

## Features

#### Tagging

Tags is used to control the invalidation of items. 

```php
$item = $pool->getItem('tobias');
$item->set('value')->setTags(['tag0', 'tag1'])
$pool->save($item);

$item = $pool->getItem('aaron');
$item->set('value')->setTags(['tag0']);
$pool->save($item);

// Remove everything tagged with 'tag1'
$pool->invalidateTags(['tag1']);
$pool->getItem('tobias')->isHit(); // false
$pool->getItem('aaron')->isHit(); // true

$item = $pool->getItem('aaron');
echo $item->getPreviousTags(); // array('tag0')

// No tags will be saved again. This is the same as saving
// an item with no tags.
$pool->save($item);
```

#### Hierarchy

Think of a hierarchy like a file system. If you remove a folder "Foo", all items and folders in "Foo" will also be removed. 
A hierarchical cache key must start with a pipe ("|").


```php
$pool->hasItem('|users|4711|followers|12|likes'); // True
$pool->deleteItem('|users|4711|followers');
$pool->hasItem('|users|4711|followers|12|likes'); // False
```

#### Namespace

Namespace can be used to separate the storage of different systems in the cache. This allows different sections to be cleared
on an individual level, while also preventing overlapping keys.

```php
$pool = new ArrayCachePool();

$namespaceFoo = new NamespacedCachePool($pool, 'foo');
$item = $namespaceFoo->getItem('key')->set('value');
$namespaceFoo->save($item);

$namespaceBar = new NamespacedCachePool($pool, 'bar');
$namespaceBar->hasItem('key'); // False
$item = $namespaceBar->getItem('key')->set('value');
$namespaceBar->save($item);

$namespaceBar->hasItem('key'); // True
$namespaceFoo->deleteItem('key');
$namespaceFoo->hasItem('key'); // False
$namespaceBar->hasItem('key'); // True

$namespaceFoo->clear();
$namespaceBar->hasItem('key'); // True
```

#### Prefix

A prefix will help you to avoid cache key collisions. The prefixed cache pool supports any PSR-6 
cache implementations. The PrefixedCachePool differs from the NamespacedCachePool in two aspects: 

1) You could still have conflicts if one cache key includes the prefix
2) When clearing the cache all cache items will be cleared, not only the prefixed ones. 

```php
$pool = new ArrayCachePool();

$prefixedFoo = new PrefixedCachePool($pool, 'foo');
$item = $prefixedFoo->getItem('key')->set('value');
$prefixedFoo->save($item);

$pool->hasItem('key'); // False
$item = $pool->getItem('key')->set('value');
$pool->save($item);

$pool->hasItem('key'); // True
$prefixedFoo->deleteItem('key');
$prefixedFoo->hasItem('key'); // False
$pool->hasItem('key'); // True

$prefixedFoo->clear();
$pool->hasItem('key'); // False
```


## Framework integration

#### Symfony

There are two Symfony bundles; [AdapterBundle] and [CacheBundle]. 

The AdapterBundle is used to configure and register a PSR-6 cache pool as a Symfony service. The  CacheBundle is used to 
integrate **any** PSR-6 cache service with the framework. It supports session cache, doctrine cache, validation cache and 
many more. 

We would LOVE to see integration with Zend, Laravel, Yii, Cake, and even Code Igniter. If you would like to contribute, 
we would love to see your code.

## Organisation overview

Excluding our adapters, we have the following packages

| Name | Description | Badges |
| ---- | ----------- | ------ |
| [Cache] | Base Cache Repository. Contains all adapters. | [![Latest Stable Version](https://poser.pugx.org/cache/cache/v/stable)](https://packagist.org/packages/cache/cache) [![Total Downloads](https://poser.pugx.org/cache/cache/downloads)](https://packagist.org/packages/cache/cache)
| [AdapterBundle] | Bundle to register adapters to services. | [![Latest Stable Version](https://poser.pugx.org/cache/adapter-bundle/v/stable)](https://packagist.org/packages/cache/adapter-bundle) [![Total Downloads](https://poser.pugx.org/cache/adapter-bundle/downloads)](https://packagist.org/packages/cache/adapter-bundle)
| [Adapter common] | The `AbstractCachePool` and `CacheItem` live here. | [![Latest Stable Version](https://poser.pugx.org/cache/adapter-common/v/stable)](https://packagist.org/packages/cache/adapter-common) [![Total Downloads](https://poser.pugx.org/cache/adapter-common/downloads)](https://packagist.org/packages/cache/adapter-common)
| [CacheBundle] | Bundle to integrate **any** PSR-6 service with the<br>Symfony framework. | [![Latest Stable Version](https://poser.pugx.org/cache/cache-bundle/v/stable)](https://packagist.org/packages/cache/cache-bundle) [![Total Downloads](https://poser.pugx.org/cache/cache-bundle/downloads)](https://packagist.org/packages/cache/cache-bundle)
| [Doctrine bridge] | A bridge from PSR-6 to DoctrineCache | [![Latest Stable Version](https://poser.pugx.org/cache/psr-6-doctrine-bridge/v/stable)](https://packagist.org/packages/cache/psr-6-doctrine-bridge) [![Total Downloads](https://poser.pugx.org/cache/psr-6-doctrine-bridge/downloads)](https://packagist.org/packages/cache/psr-6-doctrine-bridge)
| [Hierarchical cache] | A trait and interface to support cache hierachy | [![Latest Stable Version](https://poser.pugx.org/cache/hierarchical-cache/v/stable)](https://packagist.org/packages/cache/hierarchical-cache) [![Total Downloads](https://poser.pugx.org/cache/hierarchical-cache/downloads)](https://packagist.org/packages/cache/hierarchical-cache)
| [Integration tests] | Used to verify **any** PSR-6 implementation | [![Latest Stable Version](https://poser.pugx.org/cache/integration-tests/v/stable)](https://packagist.org/packages/cache/integration-tests) [![Total Downloads](https://poser.pugx.org/cache/integration-tests/downloads)](https://packagist.org/packages/cache/integration-tests)
| [Namespaced cache] | Pool to support a namespace | [![Latest Stable Version](https://poser.pugx.org/cache/namespaced-cache/v/stable)](https://packagist.org/packages/cache/namespaced-cache) [![Total Downloads](https://poser.pugx.org/cache/namespaced-cache/downloads)](https://packagist.org/packages/cache/namespaced-cache)
| [Prefixed cache] | Pool to support a prefix | [![Latest Stable Version](https://poser.pugx.org/cache/prefixed-cache/v/stable)](https://packagist.org/packages/cache/prefixed-cache) [![Total Downloads](https://poser.pugx.org/cache/prefixed-cache/downloads)](https://packagist.org/packages/cache/prefixed-cache)
| [Session handler] | Implementation of `\SessionHandlerInterface` | [![Latest Stable Version](https://poser.pugx.org/cache/session-handler/v/stable)](https://packagist.org/packages/cache/session-handler) [![Total Downloads](https://poser.pugx.org/cache/session-handler/downloads)](https://packagist.org/packages/cache/session-handler)
| [Simple Cache Bridge] | Bridge from PSR-6 to PSR-16 SimpleCache | [![Latest Stable Version](https://poser.pugx.org/cache/simple-cache-bridger/v/stable)](https://packagist.org/packages/cache/simple-cache-bridge) [![Total Downloads](https://poser.pugx.org/cache/simple-cache-bridge/downloads)](https://packagist.org/packages/cache/simple-cache-bridge)
| [Taggable cache] | Traits and interfaces to support cache tagging | [![Latest Stable Version](https://poser.pugx.org/cache/taggable-cache/v/stable)](https://packagist.org/packages/cache/taggable-cache) [![Total Downloads](https://poser.pugx.org/cache/taggable-cache/downloads)](https://packagist.org/packages/cache/taggable-cache)

## Contact

[![Gitter](https://badges.gitter.im/php-cache/cache.svg)](https://gitter.im/php-cache/cache) 

We would love to hear form you. Ping us on twitter [@aequasi](https://twitter.com/aequasi) and [@tobiasnyholm](https://twitter.com/tobiasnyholm). 
You could also join us on [Gitter](https://gitter.im/php-cache/cache).

[AdapterBundle]: https://github.com/php-cache/adapter-bundle
[Adapter common]: https://github.com/php-cache/adapter-common
[Apc]: https://github.com/php-cache/apc-adapter
[Apcu]: https://github.com/php-cache/apcu-adapter
[Array]: https://github.com/php-cache/array-adapter
[Cache]: https://github.com/php-cache/cache
[CacheBundle]: https://github.com/php-cache/cache-bundle
[Chain]: https://github.com/php-cache/chain-adapter
[Doctrine]: https://github.com/php-cache/doctrine-adapter
[Doctrine bridge]: https://github.com/php-cache/doctrine-bridge
[Filesystem]: https://github.com/php-cache/filesystem-adapter
[Illuminate]: https://github.com/php-cache/illuminate-adapter
[Hierarchical cache]: https://github.com/php-cache/hierarchical-cache
[Integration tests]: https://github.com/php-cache/integration-tests
[Memcache]: https://github.com/php-cache/memcache-adapter
[Memcached]: https://github.com/php-cache/memcached-adapter
[MongoDB]: https://github.com/php-cache/mongodb-adapter
[Predis]: https://github.com/php-cache/predis-adapter
[Redis]: https://github.com/php-cache/redis-adapter
[Namespaced cache]: https://github.com/php-cache/namespaced-cache
[Prefixed cache]: https://github.com/php-cache/prefixed-cache
[Session handler]: https://github.com/php-cache/session-handler
[Simple Cache Bridge]: https://github.com/php-cache/simple-cache-bridge
[Taggable cache]: https://github.com/php-cache/taggable-cache
[Void]: https://github.com/php-cache/void-adapter
[Flysystem]: http://flysystem.thephpleague.com/
