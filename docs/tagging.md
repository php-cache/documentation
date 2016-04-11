# Tagging your cache items

## Usage

To use Tagging with your project, use a pool that implements the `TaggablePoolInterface`, and follow the code examples below.
We create three cache items and store them in the cache with different tags. 

```php
// $pool is a PSR-6 cache that implements TaggablePoolInterface

$item = $pool->getItem('tobias');
$item->set('value')
  ->setTags(['tag0', 'tag1'])
$pool->save($item);

$item = $pool->getItem('aaron');
$item->set('value')
  ->addTag('tag0');
$pool->save($item);
```

You can clear the cache like so:

```php

// Remove everything tagged with 'tag1'
$pool->clearTags(['tag1']);
$pool->getItem('tobias')->isHit(); // false
$pool->getItem('aaron')->isHit(); // true

// To clear everything
$pool->clear();
```


# Tagging in third party libraries

If you are writing a library and you need to use tagging you should not type hint for the `TaggablePoolInterface` because that will force the 
users of your library to use php-cache. You should still type hint to `CacheItemPoolInterface` but you should use our `TaggablePSR6PoolAdapter`
that will work as a decorator for `CacheItemPoolInterface` so you can use it as a `TaggablePoolInterface`. 


```php
class AcmeLibrary
{
    /**
     * @var TaggablePoolInterface
     */
    private $cache;
    
    public function __construct(CacheItemPoolInterface $cache) {
        $this->cache = TaggablePSR6PoolAdapter::makeTaggable($cache);
    }
}
```
If the user uses a `TaggablePoolInterface` to construct `AcmeLibrary` this will have no overhead. 
