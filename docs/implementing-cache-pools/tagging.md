# Use cache tags


## Implementation

If you are writing a PSR-6 implementation you may want to use this library. The implementation is easy and will work
with all PSR-6 caches. 

You need to do a few changes on the implementation of both `CacheItemPoolInterface` and `CacheItemInterface`. 

* Implement `TaggableItemInterface`
* Implement `TaggablePoolInterface` and use `TaggablePoolTrait`

The `TaggablePoolInterface` uses exclamation mark (!) internally to be able to support tagging. Becasue exclamation is 
not a supported PSR-6 cache key chars we will not fear that the user accidently writes to a tag key but you have to 
make sure that the storage supports this character. 

### Implement TaggableItemInterface

The `TaggableItemInterface` has three methods to access the tags. They are `getTags()`, `setTags(array)`, `addTag()`.

```php
class PoolItem implements TaggableItemInterface
{
    private $tags = [];
  
    public function getTags()
    {
        return $this->tags;
    }

    public function addTag($tag)
    {
        $this->tags[] = $tag;

        return $this;
    }

    public function setTags(array $tags)
    {
        $this->tags = $tags;

        return $this;
    }
}
```

### Implement TaggablePoolInterface and use TaggablePoolTrait

The `TaggablePoolInterface` is very simple it has only one public method which is `clearTags`. We do also have the `getItem` method in this 
interface but that has no real effect except for the return type in the doc block. 

The `TaggablePoolTrait` might make it easier to implement a correct taggable cache pool. It has four methods to manipulate the list of cache keys
grouped by tag (`getList()`, `removeList()`, `appendListItem()` and `removeListItem()`). This list must be stored in the cache storage. The trait does
also implement the `clearTags()` method. 

When you delete an item with tags you have to make sure to remove the item key from the list. So in your `deleteItem()` and `deleteItems()` you must
call `TaggablePoolTrait::preRemoveItem()`. 


## Test your tagging cache

When you are happy with your implementation you should test it. We have provided some integration tests that will test
your implementation for you. See the [related documentation](integration-tests.md).
