# PSR-6 Cache Introduction
 
In the late fall 2015 the [PHP-FIG](http://www.php-fig.org/) released a specification
for caching. It is called [PSR-6](http://www.php-fig.org/psr/psr-6/). Like all the 
PSRs, it is just a recommendation for improve interoperability. 

## Basic concepts

The PSR-6 defines two interfaces; a CachePool and a CacheItem. You interact with the 
pool to get an item. You will never create an item yourself. 

```php
// Create a pool
$pool = new ApcuCachePool();

// Get an item (existing or new)
$item = $pool->getItem('cache_key');

// Set some values and store
$item->set('value');
$item->expiresAfter(60);
$pool->save($item);

// Verify existence
$pool->hasItem('cache_key'); // True
$item->isHit(); // True

// Delete
$pool->deleteItem('cache_key');
$pool->hasItem('cache_key'); // False
```
 
 
*Note: A cache item can not be transferred from one pool to another.*

## Cache keys

According to PSR-6 the following characters could be used in a valid cache key; 
`A-Z`, `a-z`, `0-9`, `_`, and `.`. Some characters are forbidden like: `{}()/\@:`. If
you use any of the forbidden characters you will get an exception. Other characters
(like `-`) are not forbidden nor valid. It is up to the implementation if they support
that character or not. 

**We recommend to always use valid characters in the cache key.**

To make sure you do not use an invalid character by mistake, you should hash your keys. 
 
```php
$cacheKey = sha1($_SERVER['REQUEST_URI']);
```

## Choosing a pool

What cache pool you  want to use is up to you. As you can see in the 
[table on the startpage](index.md#cache-pool-implementations), different pools have
different features. Commonly used poos are: 

* Apcu
* Array
* Memcached
* Redis

The steps to create a pool varies from pool to pool. Look at the pool's repository
to see how to create it. Generally you create an connection to a cache implementation
and then give it to the pool. See a Memcached example: 

```php
$client = new \Memcached();
$client->addServer('localhost', 11211);
$pool = new MemcachedCachePool($client);
```


## Features

The PHP-Cache organization has built some more features on top of PSR-6. Look at our
documentation for [tagging](tagging.md), [hierarchy](hierarchy.md) or 
[namespace](namespace.md) for more information.
