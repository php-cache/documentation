# Documentation for PHP Cache

PHP Cahe is an organisation that thinks that cache should be PSR-6 and light weight. 



## Cache pool implementations
There are plenty of adapters in this organisaion. Each of them lives in a different repository. Splitting them up in multiple packages complies with the *Common resue principle* and makes it easier for the developer to follow the changes of a specific adapter. 

Each adapter has it own features. The table below list all our adapters and their features. 


| Adapter | Tagging | Hierarcy | Removes stalled |
| ------- | ------- | -------- | --------------- |
| Apc | Yes | No  | No
| Apcu | Yes | No | No
| Array | Yes | No | No
| Couchbase (via Doctrine)| Yes | No | ??
| Filesystem (via Flysystem) | Yes | No | No
| Memcache | Yes | No | Yes
| Memcached | Yes | No | Yes
| MongoDB | Yes | No | No
| Predis| Yes | Yes | Yes
| Redis | Yes | No | Yes
| Riak (via Doctrine)| Yes | No | ??
| SQLite3 (via Doctrine)| Yes | No | No
| Void| Yes | No | No
| WinCache (via Doctrine)| Yes | No | ??
| Xcache (via Doctrine)| Yes | No | ??
| ZendData (via Doctrine)| Yes | No | ??

### Chain adapter

We also have a chain adapter where you can chain multiple pool together. It is great if you got a fast storage with limited memory and a slower storage with loads of memory. 

## Framework integration

### Symfony

We have two Symfony bundles. [AdapterBundle](https://github.com/php-cache/adapter-bundle) which you use to configure and register any PSR-6 cache pool as a Symfony service. The other bundle is called [CacheBundle](https://github.com/php-cache/cache-bundle) which you use to integrate a PSR-6 cache service with the framework. It supports session cache, doctrine cache, validation cache and many more. 
