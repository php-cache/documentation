# Cache-bundle 

This is a Symfony bundle that lets you integrate **any** PSR-6 compliant cache service with the framework. 
It lets you cache your sessions, routes, Doctrine results and metadata. It also provides an integration with the 
debug toolbar. 

When using this bundle you may also be interested in [AdapterBundle](adapter-bundle.md) which will help you configure 
and register PSR-6 cache pools as services.

## To Install

Run the following in your project root, assuming you have composer set up for your project

```sh
composer require cache/cache-bundle
```

Add the bundles to app/AppKernel.php

```php
$bundles = [
    // ...
    new Cache\CacheBundle\CacheBundle(),
];
```


## Configuration

Below is configuration for the different components where you can use caching. Most components has a `use_tagging` 
option. That will add tags to the cache item which makes it possible to be selective when clearing cache. 

To see all the config options, run `php app/console config:dump-reference cache`.

### Doctrine

This bundle allows you to use its services for Doctrine's caching methods of metadata, result and query. To use this 
feature you need to install the [DoctrineBridge]. 

```sh
composer require cache/psr-6-doctrine-bridge
```


If you want Doctrine to use this as the result and query cache, you need this configuration: 

```yaml
cache:
  doctrine:
    enabled: true
    use_tagging: true
    metadata:
      service_id: 'cache.provider.acme_redis_cache'
      entity_managers:   [ default ]       # the name of your entity_manager connection
      document_managers: [ default ]       # the name of your document_manager connection
    result:
      service_id: 'cache.provider.acme_redis_cache'
      entity_managers:   [ default, read ] # you may specify multiple entity_managers
    query:
      service_id: 'cache.provider.acme_redis_cache'
      entity_managers: [ default ]
```

To use this with Doctrine's entity manager, just make sure you have `useResultCache` and/or `useQueryCache` set to true. 

```php
$em = $this->get('doctrine.orm.entity_manager');
$q = $em->('SELECT u.* FROM Acme\User u');
$q->useResultCache(true, 3600); 
$result = $q->getResult();

```

### Session

This bundle even allows you to store your session data in one of your cache clusters. To enable:

```yml
cache:
  session:
    enabled: true
    service_id: 'cache.provider.acme_redis_cache'
    use_tagging: true
    ttl: 7200
```

### Router

This bundle also provides router caching, to help speed that section up. To enable:

```yml
cache:
  router:
    enabled: true
    service_id: 'cache.provider.acme_redis_cache'
    ttl: 86400
```

The routing cache will make the route lookup more performant when your application have many routes, especially many 
dynamic routes. If you just have a few routes your performance will actually be worse by enabling this. 
Use [Blackfire](https://blackfire.io/) to profile your application to see if you should enable routing cache or not. 

If you change any of your routes, you will need to clear the cache. If you use a cache implementation that supports 
tagging (implements [TaggablePoolInterface](https://github.com/php-cache/taggable-cache/blob/master/src/TaggablePoolInterface.php)) 
you can clear the cache tagged with `routing`.

### Logging

If you want to log all the interaction with the cache, you may do so with the following configuration.

```yml
cache:
  logging:
    enabled: true
    logger: 'logger' # Default service id to use for logging
    level: 'info' # Default logging level
```

### Annotation

To use a PSR-6 cache for your annotations, use the following configuration.

```yml
cache:
  annotation:
    enabled: true
    service_id: 'cache.provider.acme_apc_cache'
    use_tagging: true
    
framework:
  annotations:
    cache: 'cache.service.annotation'
```

### Serialization

To use a PSR-6 cache for the serializer, use the following configuration. 

```yml
cache:
  serializer:
    enabled: true
    service_id: 'cache.provider.acme_apc_cache'
    use_tagging: true
    
framework:
  serializer:
    cache: 'cache.service.serializer'
```

### Validation

To use a PSR-6 cache for the validation, use the following configuration. 

```yml
cache:
  validation:
    enabled: true
    service_id: 'cache.provider.acme_apc_cache'
    use_tagging: true

framework:
  validation:
    cache: 'cache.service.validation'
```


## Clearing the cache

If you want to clear the cache you can run the following commands.

```sh
php app/console cache:flush session
php app/console cache:flush router
php app/console cache:flush doctrine

echo "This is the same as php app/console cache:clear"
php app/console cache:flush symfony 

echo "Or you could run:"
php app/console cache:flush all

echo "Run the following command to see all your options:"
php app/console cache:flush help
```

*Caution: If you are using an implementation that does not support tagging you will clear all with any of the above commands. 
Make sure you always have the option `use_tagging` to avoid this.* 


[CacheAdapterBundle]:https://github.com/php-cache/cache-adapter-bundle
[DoctrineBridge]:https://github.com/php-cache/doctrine-bridge
