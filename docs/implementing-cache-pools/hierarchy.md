# Hierarchical PSR-6 cache pool 

*Note: Performance will be best with a driver such as memcached or redis, which automatically purges stale records.*

The trait for hierarchy pools creates a form of tag for each level in the hierarchy. That means that for every *input 
key* there are multiple *path keys*. The *path keys* are used to fetch a path value in the storage wich is used to 
create a storage key for the `CacheItem`. The idea is complex but it is easy to use. Consider this example: 

```php
// Input key is form the user
$inputKey = '|foo|bar';
$pathKey1 = 'path!foo';
$pathKey2 = 'path!foo![foo_idx]!bar';
$storageKey = 'foo![foo_idx]!bar![bar_idx]';

$this->storage->set($storageKey, $item);
```

To clear the input key `|foo` you need to update `foo_idx` simply by increasing the value by one. The implementation
becomes a bit more complex when you add support for tags but luckily there is a trait to help you with this. 

To implement hierarchy cache there are four things you need to do: 

* Implement `HierarchicalPoolInterface` and use `HierarchicalCachePoolTrait`
* Use `HierarchicalCachePoolTrait::getHierarchyKey($key)`
* Delete items better
* Implement `HierarchicalCachePoolTrait::getValueFormStore($key)`


## Implmenent the interface and use the trait
 
The trait has two protected functions `getHierarchyKey($inputKey)` and `clearHierarchyKeyCache()` and one abstract
 function `getValueFormStore($key)`. We will talk about those later. 
 
```php

use Cache\Hierarchy\HierarchicalCachePoolTrait;
use Cache\Hierarchy\HierarchicalPoolInterface;

class MyCachePool extends AbstractCachePool implements HierarchicalPoolInterface
{
  use HierarchicalCachePoolTrait;
    
  // ..
}
```

## Use HierarchicalCachePoolTrait::getHierarchyKey

To convert an input key to a storage key you need to run `HierarchicalCachePoolTrait::getHierarchyKey($key)`. It will 
create the storage key for you. You could give in a tagged key like `foo:tagHash`. 

```php
public function getItem($key)
{
  $storageKey = $this->getHierarchyKey($key);

  // ...
}
```

You should do this for the followong functions: 

* getItem
* getItems
* hasItem
* clear
* deleteItem
* deleteItems


# Delete items better

When you are clearing the cache there are quite a few things to think about. You need to update the stored value for 
the path key and also clear the local hierarcy cache key storage. To get a path key you can use the second argument of
`HierarchicalCachePoolTrait::getHierarchyKey($inputKey, $pathKey)` that is passed by reference. 

```php
public function deleteItem($key)
{
  // Get storage key and path key
  $pathKey = null;
  $storageKey = $this->getHierarchyKey($key, $pathKey);
  
  // Update index for the path key
  $pathIndex = $this->storage->get($pathKey);
  $this->storage->set($pathKey, $pathIndex + 1);
  
  // Clear the local key cache
  $this->clearHierarchyKeyCache();
  
  // ..
  
  $this->storage->delete($storageKey);
}
```

## Implement getValueFormStore($key)

The trait has one abstract function `HierarchicalCachePoolTrait::getValueFormStore($key)` wich is used to get the 
value for a path key. This functions should just feth data from the storage without modifications. 

```php
 protected function getValueFormStore($key)
 {
   return $this->storage->get($key);
 }
 ```