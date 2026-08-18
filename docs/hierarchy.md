# Hierarchical PSR-6 cache pools

Hierarchical keys let an app invalidate a branch of related data with one deletion. They start with the `|` separator.

The following example stores one item for each follower:

```php
$userId = 42;

for ($followerId = 0; $followerId < 100; ++$followerId) {
    $key = sprintf('|users|%d|followers|%d|likes', $userId, $followerId);
    $item = $pool->getItem($key);
    $item->set(12);
    $pool->save($item);
}
```

Deleting the followers branch invalidates every item below it:

```php
$pool->deleteItem('|users|42|followers');

$pool->hasItem('|users|42|followers|7|likes');
```

The final call returns `false`. Items outside the deleted branch remain available.

Hierarchy uses internal path records. Backends that evict stale records automatically, such as Redis and Memcached, are a good fit for long-running applications.

The Array, Illuminate, Memcached, Predis, Redis, and Void adapters implement `HierarchicalPoolInterface`.
