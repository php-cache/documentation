# Adapter Bundle

This bundle helps you configure and register PSR-6 cache services. 

## To Install

Run the following in your project root, assuming you have composer set up for your project

```sh
composer require cache/adapter-bundle
```

Add the bundle to app/AppKernel.php

```php
$bundles = [
    // ...
    new Cache\AdapterBundle\CacheAdapterBundle(),
];
```

## Configuration

To configure and register a cache pool adapter you need a factory service. You may also need to change some options to 
the factory. Below is the template which all adapter follow. 

```yaml
cache_adapter:
  providers:
    my_adapter:
      factory: 'service_id.to.my_adapter_factory'
      options: []
```

The factories that come with this bundle can be found in the table below. 

| Factory service id | Options | 
| ------------------ | ------- |
| cache.factory.apc |  |
| cache.factory.apcu |  |
| cache.factory.array |  |
| cache.factory.filesystem | `flysystem_service` |
| cache.factory.memcached | `host`, `port` |
| cache.factory.memcache | `host`, `port` |
| cache.factory.mongodb | `host`, `port`, `namespace` |
| cache.factory.predis | `host`, `port`, `schema` |
| cache.factory.redis | `host`, `port`, `namespace` |
| cache.factory.void |  |
| | |
| cache.factory.doctrine_couchbase | `host`, `user`, `password`, `bucket` |
| cache.factory.doctrine_filesystem | `directory`, `extension`, `umask` |
| cache.factory.doctrine_memcached | `host`, `port` |
| cache.factory.doctrine_memcache | `host`, `port` |
| cache.factory.doctrine_mongodb | `host`, `collection` |
| cache.factory.doctrine_predis | `host`, `port`, `schema` |
| cache.factory.doctrine_redis | `host`, `port` |
| cache.factory.doctrine_riak | `host`, `port`, `type` |
| cache.factory.doctrine_sqlite3 | `file_path`, `table` |
| cache.factory.doctrine_wincache |  |
| cache.factory.doctrine_xcache |  |
| cache.factory.doctrine_zenddata |  |


### Example configuration

```yaml
cache_adapter:
  providers:
    my_redis:
      factory: 'cache.factory.redis'
      options: 
        host: 10.0.0.15 # Using custom host
        port: 6379
    my_memcached:
      factory: 'cache.factory.memcached'
    my_file_system:
      factory: 'cache.factory.filesystem'
      options:
        flysystem_service: 'oneup_flysystem.local_filesystem'
    my_apc:
      factory: 'cache.factory.apc' 
```


### Usage

When using the example configuration above, you will get four services with the ids `cache.provider.my_redis`,
 `cache.provider.my_memcached`, `cache.provider.my_file_system` and `cache.provider.my_apc`.

Use the new service as any PSR-6 cache. 
 
``` php
/** @var CacheItemPoolInterface $pool */
$pool = $this->container->get('cache.provider.my_memcached');

/** @var CacheItemInterface $item */
$item = $pool->getItem('cache-key');
$item->set('foobar');
$item->expiresAfter(3600);
$pool->save($item);
```

The first configured adapter or a adapter named `default` will be aliased to service id `cache`.

``` php
// This will be an alias for 'cache.provider.my_redis'
$cache = $this->container->get('cache');
```